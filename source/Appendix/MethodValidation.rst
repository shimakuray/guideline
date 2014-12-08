Method Validationを使用した契約プログラミング
================================================================================

.. only:: html

 .. contents:: 目次
    :depth: 3
    :local:

.. _MethodValidationProgrammingByContract:

契約プログラミングとは
--------------------------------------------------------------------------------

契約プログラミングとは、プログラムが満たすべき仕様(契約)が満たされているかをチェックするためのロジックを、
プログラム内に組み込むことで、プログラムの安全性を高める事を目的としたプログラミング技法である。

具体的には、

* メソッドの引数の妥当性チェック
* メソッドの返り値の妥当性チェック

を実装することをさす。

契約プログラミングでは、

* メソッドの呼び出し前に保証すべき条件の事を **「事前条件」** と呼び、メソッドを呼び出す側がメソッドの引数に渡す値の妥当性をチェック

* メソッドの終了時に保証すべき条件の事を **「事後条件」** と呼び、メソッドを提供する側が返り値として返却する値の妥当性をチェック

するものとして、定義している。

|

.. _MethodValidationOnBeanValidation:

Bean ValidationのMethod Validation
--------------------------------------------------------------------------------

Bean Validation 1.1 より、コンストラクタとメソッドのシグネチャ(引数と返り値)に対して、
値の妥当性をチェックする仕組みが追加された。(以降「Method Validation」と呼ぶ)。

この仕様追加により、Java言語における契約プログラミングが実装しやすくなった。

Method Validationの仕組みを使用すると、

* メソッドのシグネチャ(引数と返り値)にBean Validationの制約アノテーションを付与するだけで、事前条件と事後条件のチェックを行う事ができる

* メソッドのシグネチャ(引数と返り値)にBean Validationの制約アノテーションが付与するため、特別なことをすることなく、事前条件と事後条件をJavaDoc(API仕様書)に出力する事ができる

ため、契約プログラミングの採用コストを抑えることができる。

.. note::

    Bean Validationの使用方法については、「:doc:`../ArchitectureInDetail/Validation`」も合わせて参照されたい。

|

.. _MethodValidationOnSpringFramework:

Spring FrameworkにおけるMethod Validation
--------------------------------------------------------------------------------

Spring Framework では、Bean Validationの機能と連携し、
Spring FrameworkのDIコンテナで管理されているBeanのメソッド呼び出しに対して、
透過的にMethod Validationを実行する仕組みを提供している。

そのため、メソッド内のロジックで契約プログラミングを意識した実装を一切行う必要がない。
これは、既に実装されているプログラムに対して、Bean Validationの制約アノテーションを付与するだけで、
契約プログラミングを組み込むことが出来ることを意味している。

.. tip::

    Spring Frameworkでは、Bean Validation 1.1 がリリースされる以前から、
    Hibernate Validator 4.3(Bean Validation 1.0のリファレンス実装)の機能と連携して、
    Method Validationを実行する仕組みを提供していた。

    Spring Framework 4 からは、下位互換性を保ちつつ、
    Bean Validation 1.1 の機能と連携したMethod Validationの仕組みを提供している。

.. note::

    Spring Frameworkが提供しているMethod Validationの仕組みは、メソッドの呼び出し時のみに実行される点を補足しておく。


|

.. _MethodValidationOnSpringFrameworkHowToUse:

Method Validationの使用方法
--------------------------------------------------------------------------------

ここからは、Spring Frameworkが提供するMethod Validationの具体的な使い方について説明していく。

.. _MethodValidationOnSpringFrameworkHowToUseSettings:

アプリケーションの設定
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Spring Frameworkが提供するMethod Validationを使用する場合は、
Spring Frameworkから提供されている\ ``org.springframework.validation.beanvalidation.MethodValidationPostProcessor``\ クラスをBean定義する必要がある。

\ ``MethodValidationPostProcessor``\ を定義するBean定義ファイルは、Method Validationを使用する箇所によって異なる。

ここでは、本ガイドラインが推奨するマルチプロジェクト環境下において、

* アプリケーション層用のプロジェクト(\ ``projectName-web``\ )
* ドメイン層用のプロジェクト(\ ``projectName-domain``\ )

の両プロジェクトでMethod Validationを使用する際の設定例を示す。

* :file:`projectName-domain/src/main/resources/META-INF/spring/projectName-domain.xml`

 .. code-block:: xml

    <!-- (1) -->
    <bean id="validator"
          class="org.springframework.validation.beanvalidation.LocalValidatorFactoryBean"/>

    <!-- (2) -->
    <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
        <property name="validator" ref="validator" />
    </bean>

* :file:`projectName-web/src/main/resources/META-INF/spring/spring-mvc.xml`

 .. code-block:: xml

    <!-- (3) -->
    <mvc:annotation-driven validator="validator">
        <!-- ... -->
    </mvc:annotation-driven>

    <!-- (4) -->
    <bean class="org.springframework.validation.beanvalidation.MethodValidationPostProcessor">
        <property name="validator" ref="validator" />
    </bean>


.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``LocalValidatorFactoryBean``\ をBean定義する。
    * - | (2)
      - \ ``MethodValidationPostProcessor``\ をBean定義し、
        ドメイン層のクラスのメソッドに対してMethod Validationが実行されるようにする。

        \ ``validator``\ プロパティには、(1)で定義したBeanを指定する。
    * - | (3)
      - \ ``<mvc:annotation-driven>``\ 要素の\ ``validator``\ 属性に、(1)で定義したBeanを指定する。
    * - | (4)
      - \ ``MethodValidationPostProcessor``\ をBean定義し、
        アプリケーション層のクラスのメソッドに対してMethod Validationが実行されるようにする。

        \ ``validator``\ プロパティには、(1)で定義したBeanを指定する。

.. tip::

    \ ``LocalValidatorFactoryBean``\ は、
    Bean Validation(Hibernate Validator)が提供する\ ``Validator``\ クラスとSpring Frameworkを連携するためのラッパー\ ``Validator``\ オブジェクトを生成するためのクラスである。

    このクラスによって生成されたラッパー\ ``Validator``\を使用することで、
    Spring Frameworkが提供するメッセージ管理機能(\ ``MessageSource``\ )やDIコンテナなどとの連携が行えるようになる。

.. tip::

    Spring Frameworkでは、DIコンテナで管理されているBeanのメソッド呼び出しに対するMethod Validationの実行を、
    AOPの仕組みを利用して行っている。

    \ ``MethodValidationPostProcessor``\ は、Method Validationを実行するためのAOPを適用するためのクラスである。

.. note::

    上記例では、各Beanの\ ``validator``\ プロパティに対して、同じ\ ``Validator``\ オブジェクト(インスタンス)を設定しているが、
    これは必ずしも必須ではない。
    ただし、特に理由がない場合は、同じオブジェクト(インスタンス)を設定しておくことを推奨する。


|

.. _MethodValidationOnSpringFrameworkHowToUseApplyTarget:

Method Validation対象のメソッドにするための定義方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

「:ref:`MethodValidationOnSpringFrameworkHowToUseSettings`」を行っただけでは、Method Validationを実行するAOPは適用されない。

Method Validationを実行するAOPを適用するためには、
インタフェース又はクラスレベルに\ ``@ org.springframework.validation.annotation.Validated``\ アノテーションを付与する必要がある。

ここでは、インタフェースにアノテーションを指定する方法を紹介する。

.. code-block:: java

    package com.example.domain.service;

    import org.springframework.validation.annotation.Validated;

    @Validated // (1)
    public interface HelloService {
        // ...
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - Method Validationの対象としたいインタフェースに、\ ``Validated``\ アノテーションを付与する。

        上記例では、\ ``HelloService``\ インタフェースの実装メソッドに対して、
        Method Validationを実行するAOPが適用される。

.. tip::

    \ ``@Validated``\ アノテーションの\ ``value``\ 属性にグループインタフェースを指定することで、
    指定したグループに属するValidationのみ実行する事も可能である。

    また、メソッドレベルに\ ``Validated``\ アノテーションを付与することで、
    メソッド毎にバリデーショングループを切り替える事も可能な仕組みとなっている。

    バリデーショングループについては、「:ref:`ValidationGroupValidation`」を参照されたい。


|

.. _MethodValidationOnSpringFrameworkHowToUseApplyRules:

事前条件と事後条件の指定方法
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bean Validationの制約アノテーションを使用して、事前条件と事後条件を指定する。

具体的には、

* メソッドの引数
* 引数に指定されたJavaBeanのフィールド

に対して事前条件を示すBean Validationの制約アノテーションを、

* メソッドの返り値
* メソッドの返り値として返却するJavaBeanのフィールド

に対して事後条件を示すBean Validationの制約アノテーションを指定する。

ここでは、インタフェースにアノテーションを指定する方法を紹介する。

.. code-block:: java

    package com.example.domain.service;

    import org.springframework.validation.annotation.Validated;

    import javax.validation.constraints.NotNull;

    @Validated
    public interface HelloService {
        /* (2) */ @NotNull String hello(/* (1) */  @NotNull String message);
    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 事前条件(Bean Validationの制約アノテーション)を、メソッドの引数アノテーションとして指定する。

        上記例では、事前条件として、\ ``message``\ という引数がNull値を許可しない事を示している。
    * - | (2)
      - 事後条件(Bean Validationの制約アノテーション)を、メソッドの返り値アノテーションとして指定する。

        上記例では、事後条件として、返り値がNull値にならないことを示している。

.. note::

    JavaBeanに対してMethod Validationを行う場合は、\ ``@javax.validation.Valid``\ アノテーションを付与する必要がある。
    \ ``@Valid``\ アノテーションを付与しないと、JavaBeanのフィールドに指定した事前条件又は事後条件がチェックされないので注意が必要である。


|

.. _MethodValidationOnSpringFrameworkHowToUseExceptionHandling:

契約違反時の例外ハンドリング
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

契約(事前条件及び事後条件)に違反した場合、\ ``javax.validation.ConstraintViolationException``\ が発生する。

\ ``ConstraintViolationException``\ が発生した場合、スタックトレースは表示されるため、違反が発生したメソッドは特定できるが、
具体的な違反内容が特定できない。

違反内容を特定するためには、\ ``ConstraintViolationException``\ をハンドリングしてログ出力を行う例外ハンドリングクラスを作成するとよい。

以下の例外ハンドリングクラスの作成例を示す。

.. code-block:: java

    package com.example.app;

    import javax.validation.ConstraintViolationException;

    import org.slf4j.Logger;
    import org.slf4j.LoggerFactory;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;

    @ControllerAdvice
    public class ConstraintViolationExceptionHandler {

        private static final Logger log = LoggerFactory.getLogger(ConstraintViolationExceptionHandler.class);

        @ExceptionHandler
        public String handleConstraintViolationException(ConstraintViolationException e){
            log.error("ConstraintViolations[\n{}\n]", e.getConstraintViolations());
            return "common/error/systemError";
        }

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - \ ``ConstraintViolationException``\ をハンドリングするための\ ``@ExceptionHandler``\ メソッドを作成する。
    * - | (2)
      - \ ``ConstraintViolationException``\ が保持している\ ``ConstraintViolation``\ の\ ``Set``\ を文字列化に変換し、ログに出力する。


.. raw:: latex

   \newpage

