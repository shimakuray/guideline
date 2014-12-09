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
プログラム内部に組み込むことで、プログラムの安全性を高める事を目的としたプログラミング技法である。

具体的には、

* メソッドの引数の妥当性チェック
* メソッドの返り値の妥当性チェック

を実装することをさす。

契約プログラミングでは、

* メソッドの呼び出し前に保証すべき条件の事を **「事前条件」** と呼び、メソッドを呼び出す側がメソッドの引数に渡す値の妥当性をチェック

* メソッドの終了時に保証すべき条件の事を **「事後条件」** と呼び、メソッドを提供する側が返り値として返却する値の妥当性をチェック

するものと定義している。

契約プログラミングを採用すると、

* メソッドの入出力仕様が不明瞭である事が原因で発生するバグを防止
* メソッド利用者側の実装ミス(メソッドの間違った使い方)を早期に発見
* メソッド提供者側の実装ミスを早期に発見

する事ができるため、プログラムの品質を効率的に高めることができるプログラミング技法でもある。

.. note:: **単体テストとの関係について**

    契約プログラミングでは、メソッドのシグネチャ(引数と返り値)に対する仕様の妥当性をチェックすることはできるが、
    メソッド内部のロジックの妥当性をチェックすることはできない。
    そのため、メソッド内部のロジックの妥当性をチェックするためには、
    JUnitなどを使用した単体テストの実施が必要となる。

|

.. _MethodValidationOnBeanValidation:

Bean ValidationのMethod Validation
--------------------------------------------------------------------------------

Bean Validation 1.1 より、コンストラクタとメソッドのシグネチャ(引数と返り値)に対して、
値の妥当性をチェックする仕組みが追加された。(以降「Method Validation」と呼ぶ)。

この仕様追加により、Java言語上での契約プログラミングが実現しやすくなった。

Method Validationの仕組みを使用すると、

* メソッドのシグネチャにBean Validationの制約アノテーションを付与するだけで、事前条件と事後条件のチェックを行う事ができる

* メソッドのシグネチャにBean Validationの制約アノテーションを付与するため、特別なことをすることなく、事前条件と事後条件をJavaDoc(API仕様書)に出力する事ができる

ため、契約プログラミングの導入コストを最小限に抑えつつ、契約プログラミングが持つ効果を得ることができる。

.. note::

    Bean Validationの基本的な使用方法については、「:doc:`../ArchitectureInDetail/Validation`」を参照されたい。

|

.. _MethodValidationOnSpringFramework:

Spring FrameworkにおけるMethod Validation
--------------------------------------------------------------------------------

Spring Framework では、Bean Validationの機能と連携し、
Spring FrameworkのDIコンテナで管理されているBeanのメソッド呼び出しに対して、
透過的にMethod Validationを実行する仕組みを提供している。

Spring Frameworkが提供するMethod Validationの仕組みを使用すると、
メソッドを提供する側とメソッドを利用する側で実装するロジックにおいて、
契約プログラミングを意識した実装を一切行う必要がない。
これは、既に実装されているプログラムに対して、Bean Validationの制約アノテーションを付与するだけで、
契約プログラミングを組み込むことが出来ることを意味している。

.. tip::

    Spring Frameworkでは、Bean Validation 1.1 がリリースされる以前から、
    Hibernate Validator 4.3(Bean Validation 1.0のリファレンス実装)の機能と連携して、
    Method Validationを実行する仕組みを提供していた。

    Spring Framework 4 からは、下位互換性を保ちつつ、
    Bean Validation 1.1 の機能と連携したMethod Validationの仕組みを提供している。

    また、Spring Frameworkが提供する `プロファイル切替機能 <http://docs.spring.io/spring/docs/4.1.2.RELEASE/spring-framework-reference/html/beans.html#beans-definition-profiles-xml>`_\ を利用することで、
    実行環境に応じてMethod Validationの実行有無を切り替えることも可能である。
    例えば、開発及び試験環境ではMethod Validationの実行を有効にし、商用環境では無効にすることも出来る。
    性能要件上問題がなければ、商用環境でもMethod Validationを実行する事で、アプリケーションの安全性を高めることを推奨する。

.. note::

    Spring Frameworkが提供しているMethod Validationの仕組みは、メソッドの呼び出し時のみに実行される点を補足しておく。
    (つまり、コンストラクタの呼び出しに対してMethod Validationを行うことはできない)

|

.. _MethodValidationRole:

Validationの分類
--------------------------------------------------------------------------------

アプリケーションで実行するValidationは、以下の2つの分類する事ができる。

* クライアントからの入力値の妥当性をチェックする「Input Validation」
* プログラム内部のメッセージ(メソッドの引数と返り値)をチェックする「Method Validation」

この2つのValidationが果たす役割は、以下の通りである。

.. tabularcolumns:: |p{0.10\linewidth}|p{0.20\linewidth}|p{0.70\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 20 70

    * - 項番
      - 分類
      - 役割
    * - | (1)
      - Input Validation
      - アプリケーションの外部仕様を満たすために行うチェックであり、クライアントからの不正なリクエストからアプリケーションを守る役割も合わせもつ。

        **アプリケーションを構築する際は、「Input Validation」は必ず行う必要がある。**
        「Input Validation」の詳細については、「:doc:`../ArchitectureInDetail/Validation`」を参照されたい。
    * - | (2)
      - Method Validation
      - メソッドのインタフェース仕様(メソッドの引数と返り値の仕様)を満たす実装になっているかをチェックすることで、
        ブログラムの不備(バグ)からアプリケーションを守る役割をもつ。

.. note::

    上記で示した通り、この2つのValidationが果たす役割は大きく異なるため、どちらか一方を行えばよいという関係ではない。

    「Method Validation」の導入については、

    * 構築するアプリケーションの構成
    * 開発体制

    などを加味して決めて頂きたい。

    例えば、

    * 開発体制が分散しており、別チームの作成したAPI(共通部品など)を呼び出す機会が多い
    * 実装者のスキルが低い

    といった状況下では、「Method Validation」を使用した契約プログラミングの導入を検討した方がよい。

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

        この設定は、Spring MVCが提供しているInput Validationを使用するために必要な設定である。
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

ここでは、インタフェースに対してアノテーションを指定する方法を紹介する。

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
* メソッドの引数に指定されたJavaBeanのフィールド

に対して事前条件を示すBean Validationの制約アノテーションを、

* メソッドの返り値
* メソッドの返り値として返却するJavaBeanのフィールド

に対して事後条件を示すBean Validationの制約アノテーションを指定する。

以下に、具体的な指定方法について説明する。
以降の説明では、インタフェースにアノテーションを指定する方法を紹介する。

.. _MethodValidationOnSpringFrameworkHowToUseApplyRulesBasicType:

基本型への指定方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

まず、メソッドのシグネチャとして基本型(プリミティブやプリミティブラッパ型など)を使用するメソッドに対して、
事前条件と事後条件を指定する方法について説明する。

ここでは、インタフェースに対してアノテーションを指定する方法を紹介する。

.. code-block:: java

    package com.example.domain.service;

    import org.springframework.validation.annotation.Validated;

    import javax.validation.constraints.NotNull;

    @Validated
    public interface HelloService {

        // (2)
        @NotNull
        String hello(
                // (1)
                @NotNull String message);

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 事前条件(Bean Validationの制約アノテーション)をメソッドの引数アノテーションとして指定する。

        上記例では、事前条件として、\ ``message``\ という引数がNull値を許可しない事を示しており、
        引数にNull値が指定された場合は、契約違反を通知する例外が発生する。
    * - | (2)
      - 事後条件(Bean Validationの制約アノテーション)をメソッドアノテーションとして指定する。

        上記例では、事後条件として、返り値がNull値にならないことを示しており、
        返り値としてNull値が返却された場合は、契約違反を通知する例外が発生する。

|

.. _MethodValidationOnSpringFrameworkHowToUseApplyRulesJavaBean:

JavaBeanへの指定方法
""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""""

次に、メソッドのシグネチャとしてJavaBeanを使用するメソッドに対して、
事前条件と事後条件を指定する方法について説明する。

ここでは、インタフェースに対してアノテーションを指定する方法を紹介する。

.. note::

    ポイントは、\ ``@javax.validation.Valid``\ アノテーションを指定するという点である。
    以下に、サンプルコード使って指定方法を詳しく説明する。

**Serviceインタフェース**

.. code-block:: java

    package com.example.domain.service;

    import org.springframework.validation.annotation.Validated;

    import javax.validation.constraints.NotNull;

    @Validated
    public interface HelloService {

        @NotNull // (3)
        @Valid   // (4)
        HelloOutput hello(
                    @NotNull // (1)
                    @Valid   // (2)
                    HelloInput input);

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (1)
      - 事前条件(Bean Validationの制約アノテーション)をメソッドの引数アノテーションとして指定する。

        上記例では、事前条件として、\ ``input``\ という引数(JavaBean)がNull値を許可しない事を示しており、
        引数にNull値が指定された場合は、契約違反を通知する例外が発生する。
    * - | (2)
      - \ ``@javax.validation.Valid``\ アノテーションをメソッドの引数アノテーションとして指定する。

        \ ``@Valid``\ アノテーションを付与する事で、引数のJavaBeanのフィールドに指定した事前条件(Bean Validationの制約アノテーション)が有効となる。
        JavaBeanに指定された事前条件を満たさない場合は、契約違反を通知する例外が発生する。
    * - | (3)
      - 事後条件(Bean Validationの制約アノテーション)をメソッドアノテーションとして指定する。

        上記例では、事後条件として、返り値のJavaBeanがNull値にならないことを示しており、
        返り値としてNull値が返却された場合は、契約違反を通知する例外が発生する。
    * - | (4)
      - \ ``@Valid``\ アノテーションをメソッドアノテーションとして指定する。

        \ ``@Valid``\ アノテーションを付与する事で、返り値のJavaBeanのフィールドに指定した事後条件(Bean Validationの制約アノテーション)が有効となる。
        JavaBeanに指定された事後条件を満たさない場合は、契約違反を通知する例外が発生する。

|

| 以下にJavaBeanの実装サンプルを紹介する。
| 基本的には、Bean Validationの制約アノテーションを指定するだけだが、JavaBeanが更にJavaBeanをネストしている場合は注意が必要になる。

**Input用のJavaBean**

.. code-block:: java

    package com.example.domain.service;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Past;
    import java.util.Date;

    public class HelloInput {

        @NotNull
        @Past
        private Date visitDate;

        @NotNull
        private String visitMessage;

        private String userId;

        // ...

    }

**Output用のJavaBean**

.. code-block:: java

    package com.example.domain.service;

    import com.example.domain.model.User;

    import java.util.Date;

    import javax.validation.Valid;
    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Past;

    public class HelloOutput {

        @NotNull
        @Past
        private Date acceptDate;

        @NotNull
        private String acceptMessage;

        @Valid // (5)
        private User user;

        // ...

    }

**Output用のJavaBean内でネストしているJavaBean**

.. code-block:: java

    package com.example.domain.model;

    import javax.validation.constraints.NotNull;
    import javax.validation.constraints.Past;
    import java.util.Date;

    public class User {

        @NotNull
        private String userId;

        @NotNull
        private String userName;

        @Past
        private Date dateOfBirth;

        // ...

    }

.. tabularcolumns:: |p{0.10\linewidth}|p{0.90\linewidth}|
.. list-table::
    :header-rows: 1
    :widths: 10 90

    * - 項番
      - 説明
    * - | (5)
      - ネストしたJavaBeanに指定している事前・事後条件(Bean Validationの制約アノテーション)を有効にする場合は、
        \ ``@Valid``\ アノテーションをフィールドアノテーションとして指定する。

        \ ``@Valid``\ アノテーションを付与する事で、ネストしたJavaBeanのフィールドに指定した事前・事後条件(Bean Validationの制約アノテーション)が有効となる。
        ネストしたJavaBeanに指定された事前・事後条件を満たさない場合は、契約違反を通知する例外が発生する。

|

.. _MethodValidationOnSpringFrameworkHowToUseExceptionHandling:

契約違反時の例外ハンドリング
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

契約(事前条件及び事後条件)に違反した場合、\ ``javax.validation.ConstraintViolationException``\ が発生する。

\ ``ConstraintViolationException``\ が発生した場合、スタックトレースから発生したメソッドは特定できるが、
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

        // (1)
        @ExceptionHandler
        public String handleConstraintViolationException(ConstraintViolationException e){
            // (2)
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

        メソッドの引数として、\ ``ConstraintViolationException``\ を受け取るようにする。
    * - | (2)
      - メソッドの引数で受け取った\ ``ConstraintViolationException``\ が保持している違反内容(\ ``ConstraintViolation``\ の\ ``Set``\ )をログに出力する。

.. note::

    \ ``@ControllerAdvice``\ アノテーションの詳細については「:ref:`application_layer_controller_advice`」を参照されたい。

.. raw:: latex

   \newpage

