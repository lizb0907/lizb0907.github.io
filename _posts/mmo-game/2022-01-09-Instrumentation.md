---
layout: post
title: Instrumentation热更限制
categories: Mmo-Game
description: Instrumentation热更限制
keywords: Instrumentation，热更
---

Instrumentation热更限制

**目录**

* TOC
{:toc}

## Instrumentation热更有什么限制?

我们知道游戏里对代码热更，会使用Instrumentation的redefineClasses(ClassDefinition... var1)方法来重定义类，

说白了就是动态修改字节码，那么该方法有什么限制吗？

```sh
The redefinition may change method bodies, the constant pool and attributes. 
The redefinition must not add, remove or rename fields or methods, change the signatures of methods, or change inheritance. 
These restrictions maybe be lifted in future versions. The class file bytes are not checked, verified and installed until 
after the transformations have been applied, if the resultant bytes are in error this method will throw an exception.
```

```sh
官方文档给出使用限制:
   不能添加、移除或重命名字段和方法，不能改变方法的签名，不能改变继承关系。
   同时留下了悬念说了这些限制可能会在未来被移除。
```


## 为什么有这些限制？从源码角度探究下（版本:OpenJdk8）

### 1.Instrumentation实现类InstrumentationImpl

```java
public void redefineClasses(ClassDefinition... definitions) throws ClassNotFoundException {
        if (!this.isRedefineClassesSupported()) {
            throw new UnsupportedOperationException("redefineClasses is not supported in this environment");
        } else if (definitions == null) {
            throw new NullPointerException("null passed as 'definitions' in redefineClasses");
        } else {
            for(int i = 0; i < definitions.length; ++i) {
                if (definitions[i] == null) {
                    throw new NullPointerException("element of 'definitions' is null in redefineClasses");
                }
            }

            if (definitions.length != 0) {
                this.redefineClasses0(this.mNativeAgent, definitions);
            }
        }
    }
```
我们看到InstrumentationImpl实现类redefinClasses方法，最终调用的是redefineClasses0()，

是一个本地方法。


### 2.InstrumentationImplNativeMethods.c

```java
/*
 * Class:     sun_instrument_InstrumentationImpl
 * Method:    redefineClasses0
 * Signature: ([Ljava/lang/instrument/ClassDefinition;)V
 */
JNIEXPORT void JNICALL Java_sun_instrument_InstrumentationImpl_redefineClasses0
  (JNIEnv * jnienv, jobject implThis, jlong agent, jobjectArray classDefinitions) {
    redefineClasses(jnienv, (JPLISAgent*)(intptr_t)agent, classDefinitions);
}
```
顺着本地方法往下跟，redefineClasses()调用的是JPLISAgent.c下的redefineClasses()


### 3.JPLISAgent.c

```sh
1.JVMTI (JVM Tool Interface)：
  是Java虚拟机提供的既Native编程接口，可以通过JVMTI获取大量JVM运行时的外部信息来处理，如线程、GC等。

2.libininstrument是一个动态链接库，JPLISAgent是实现代码的执行，它是Java代码与JVMTI之间的桥梁。
  我们调用Java代码redefineclass Java Instrumentation API，实际上，是调用libininstrument相关的代码。
```

```java
if (!errorOccurred) {
    jvmtiError  errorCode = JVMTI_ERROR_NONE;
    errorCode = (*jvmtienv)->RedefineClasses(jvmtienv, numDefs, classDefs);
    if (errorCode == JVMTI_ERROR_WRONG_PHASE) {
        /* insulate caller from the wrong phase error */
        errorCode = JVMTI_ERROR_NONE;
    } else {
        errorOccurred = (errorCode != JVMTI_ERROR_NONE);
        if ( errorOccurred ) {
            createAndThrowThrowableFromJVMTIErrorCode(jnienv, errorCode);
        }
    }
}

```
redefineclass实现这个函数比较复杂，代码很长，上面面是一个关键的代码片段。
重点是我们要关注的，它会调用jvmtiEnv.cpp的RedefineClasses()方法。

### 4.jvmtiEnv.cpp的RedefineClasses()

```java
// class_count - pre-checked to be greater than or equal to 0
// class_definitions - pre-checked for NULL
jvmtiError
JvmtiEnv::RedefineClasses(jint class_count, const jvmtiClassDefinition* class_definitions) {
//TODO: add locking
  VM_RedefineClasses op(class_count, class_definitions, jvmti_class_load_kind_redefine);
  VMThread::execute(&op);
  return (op.check_error());
} 
```

请求类被重新定义为JVM打包为vm_redefineclasses型VM_operation。

### 5.VM_RedefineClasses

```java
  // Verify that the caller provided class definition(s) that meet
  // the restrictions of RedefineClasses. Normalize the order of
  // overloaded methods as needed.
jvmtiError compare_and_normalize_class_versions(
    instanceKlassHandle the_class, instanceKlassHandle scratch_class);
```
VM redefineclass处理更复杂的功能，有一大堆流程，我们只需要关注上面的比较和规范类版本方法。

进入方法，我们发现方法里进行了大量的规范检测，下面列了几个重点检测：

```java
// Check superclasses, or rather their names, since superclasses themselves can be
// requested to replace.
// Check for NULL superclass first since this might be java.lang.Object
if (the_class->super() != scratch_class->super() &&
    (the_class->super() == NULL || scratch_class->super() == NULL ||
      the_class->super()->name() !=
      scratch_class->super()->name())) {
  return JVMTI_ERROR_UNSUPPORTED_REDEFINITION_HIERARCHY_CHANGED;
}
```
继承关系改变，报错。

```java
// Check if the number, names and order of directly implemented interfaces are the same.
// I think in principle we should just check if the sets of names of directly implemented
// interfaces are the same, i.e. the order of declaration (which, however, if changed in the
// .java file, also changes in .class file) should not matter. However, comparing sets is
// technically a bit more difficult, and, more importantly, I am not sure at present that the
// order of interfaces does not matter on the implementation level, i.e. that the VM does not
// rely on it somewhere.
Array<Klass*>* k_interfaces = the_class->local_interfaces();
Array<Klass*>* k_new_interfaces = scratch_class->local_interfaces();
int n_intfs = k_interfaces->length();
if (n_intfs != k_new_interfaces->length()) {
  return JVMTI_ERROR_UNSUPPORTED_REDEFINITION_HIERARCHY_CHANGED;
}
for (i = 0; i < n_intfs; i++) {
  if (k_interfaces->at(i)->name() !=
      k_new_interfaces->at(i)->name()) {
    return JVMTI_ERROR_UNSUPPORTED_REDEFINITION_HIERARCHY_CHANGED;
  }
}
```
接口数量、接口名不同报错。

```java
// Check if the number, names, types and order of fields declared in these classes
// are the same.
JavaFieldStream old_fs(the_class);
JavaFieldStream new_fs(scratch_class);
for (; !old_fs.done() && !new_fs.done(); old_fs.next(), new_fs.next()) {
  // access
  old_flags = old_fs.access_flags().as_short();
  new_flags = new_fs.access_flags().as_short();
  if ((old_flags ^ new_flags) & JVM_RECOGNIZED_FIELD_MODIFIERS) {
    return JVMTI_ERROR_UNSUPPORTED_REDEFINITION_SCHEMA_CHANGED;
  }
  // offset
  if (old_fs.offset() != new_fs.offset()) {
    return JVMTI_ERROR_UNSUPPORTED_REDEFINITION_SCHEMA_CHANGED;
  }
  // name and signature
  Symbol* name_sym1 = the_class->constants()->symbol_at(old_fs.name_index());
  Symbol* sig_sym1 = the_class->constants()->symbol_at(old_fs.signature_index());
  Symbol* name_sym2 = scratch_class->constants()->symbol_at(new_fs.name_index());
  Symbol* sig_sym2 = scratch_class->constants()->symbol_at(new_fs.signature_index());
  if (name_sym1 != name_sym2 || sig_sym1 != sig_sym2) {
    return JVMTI_ERROR_UNSUPPORTED_REDEFINITION_SCHEMA_CHANGED;
  }
}

// If both streams aren't done then we have a differing number of
// fields.
if (!old_fs.done() || !new_fs.done()) {
  return JVMTI_ERROR_UNSUPPORTED_REDEFINITION_SCHEMA_CHANGED;
}
```
检查这些类中声明的字段的数量、名称、类型和顺序是一样的。


省略其它各种检测。。。

省略其它各种检测。。。

省略其它各种检测。。。

## 从源码角度总结

Instrumentation重定义类时，不能添加、移除或重命名字段和方法，不能改变方法的签名，不能改变继承关系。

从源码角度来看是因为源码底层做了各种条件限制检测，条件不符合就会导致重定义失败。

## 为什么源码的作者需要加入这些限制检测？并说未来可能移除这些检测？

个人猜测，重定义类总的来说就是只更新了类里内容，相当于只更新了指针指向的内容，并没有更新指针，

当前这么做可以避免遍历大量已有类对象对它们进行更新带来的开销。（这里需要补充new对象的jvm里相关知识）


## 参考文章

https://titanwolf.org/Network/Articles/Article?AID=da87f6a7-2ba1-4407-8bfb-74ccb1e0e46d








