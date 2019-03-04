---

layout: post

title: 'Android AnnotationProcess及注解'

subtitle: 'AnnotationProcess' 

date: 2017-03-01

categories: Android札记

tags: Android Annotation AnnotationProcess

---
注解及通过AnnotationProcess编译时生成代码
---

# 注解
## 1. 什么是注解?
注解可以向编译器、虚拟机等解释说明一些事情.
Like`@override`它的作用是告诉编译器它所注解的方法是重写父类的方法，这样编译器就会去检查父类是否存在这个方法.
> 注解是用来描述Java代码的，它既能被编译器解析，也能在运行时被解析

# 2. 元注解
> 对注解进行注解的注解

## 2.1 @Documented
Javadoc工具文档化
## 2.2 @Retention
* `ElementType.SOURCE`编译时移除
* `ElementType.CLASS` 包含在class文件中,运行时移除
* `ElementType.RUNTIME`运行时移除

## 2.3 @Target
 * `ElementType.TYPE`  类、接口、注解类型或枚举类型。
 * `ElementType.PACKAGE`  注解包。
 * `ElementType.PARAMETER`  注解参数。
 * `ElementType.ANNOTATION_TYPE`  注解 注解类型。
 * `ElementType.METHOD`  方法。
 * `ElementType.FIELD`  属性（包括枚举常量）
 * `ElementType.CONSTRUCTOR`  构造器。
 * `ElementType.LOCAL_VARIABLE`  局部变量

## 2.4 @Inherited 继承
 注解会被子类继承,通过反射`clazz.getAnnotations()`可以获取父类定义有`@inherited`的注解

# 3. 注解的使用 
## 运行时注解
运行时 通过反射获取注解信息,进行操作
## 编译时注解
类似ButterKnife,Dagger,Retrofit都是利用编译时注解,通过`AnnotationProcess`,以及利用Google的`AutoService`和Square的`javapoet`来自动生成代码

[ButterKnife的简单实现](https://www.jianshu.com/p/2585d2a7cd97)

### 实现AbstractProcessor类,来实现编译时生成代码

1. 编译时，系统会调用所有AbstractProcessor子类的process方法，也就是调用我们的ViewFinderProcess的类。
在ViewFinderProcess中，我们获得工程下所有被@BindView注解所修饰的View。
1. 遍历这些被@BindView修饰的View变量，获得它们被声明时所在的类，首先判断是否已经为所在的类生成了对应的AnnotatedClass，如果没有，那么生成一个，并将View封装成BindViewField添加进入AnnotatedClass的列表，反之添加即可，所有的AnnotatedClass被保存在一个map当中。
2. 当遍历完所有被注解修饰的View后，开始遍历之前生成的AnnotatedClass，每个AnnotatedClass会生成一个对应的$$Finder类。
3. 如果我们在n个类中使用了@BindView来修饰里面的View，那么我们最终会得到n个`$$Finder`类，并且无论我们最终有没有在这n个类中调用ViewFinder.inject方法，都会生成这n个类；而如果我们调用了ViewFinder.inject，那么最终就会通过反射来实例化它对应的`$$Finder`类，通过调用inject方法来给被它里面被@BindView所修饰的View执行findViewById操作。

```java
@AutoService(Processor.class)
public class ViewFinderProcess extends AbstractProcessor{

    private Filer mFiler;
    private Elements mElementUtils;
    private Messager mMessager;

    private Map<String, AnnotatedClass> mAnnotatedClassMap = new HashMap<>();

    @Override
    public synchronized void init(ProcessingEnvironment processingEnv) {
        super.init(processingEnv);
        mFiler = processingEnv.getFiler();
        mElementUtils = processingEnv.getElementUtils();
        mMessager = processingEnv.getMessager();
    }

    @Override
    public Set<String> getSupportedAnnotationTypes() {
        Set<String> types = new LinkedHashSet<>();
        types.add(BindView.class.getCanonicalName());
        return types;
    }

    @Override
    public SourceVersion getSupportedSourceVersion() {
        return SourceVersion.latestSupported();
    }

    @Override
    public boolean process(Set<? extends TypeElement> annotations, RoundEnvironment roundEnv) {
        mAnnotatedClassMap.clear();
        try {
            processBindView(roundEnv);
        } catch (IllegalArgumentException e) {
            return true;
        }
        for (AnnotatedClass annotatedClass : mAnnotatedClassMap.values()) { //遍历所有要生成$$Finder的类.
            try {
                annotatedClass.generateFinder().writeTo(mFiler); //一次性生成.
            } catch (IOException e) {
                return true;
            }
        }
        return true;
    }

    private void processBindView(RoundEnvironment roundEnv) throws IllegalArgumentException {
        for (Element element : roundEnv.getElementsAnnotatedWith(BindView.class)) {
            AnnotatedClass annotatedClass = getAnnotatedClass(element);
            BindViewField field = new BindViewField(element);
            annotatedClass.addField(field);
        }
    }

    private AnnotatedClass getAnnotatedClass(Element element) {
        TypeElement classElement = (TypeElement) element.getEnclosingElement();
        String fullClassName = classElement.getQualifiedName().toString();
        AnnotatedClass annotatedClass = mAnnotatedClassMap.get(fullClassName);
        if (annotatedClass == null) {
            annotatedClass = new AnnotatedClass(classElement, mElementUtils);
            mAnnotatedClassMap.put(fullClassName, annotatedClass);
        }
        return annotatedClass;
    }
}
```


  




