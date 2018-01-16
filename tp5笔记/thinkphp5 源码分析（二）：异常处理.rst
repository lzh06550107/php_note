*********
异常处理
*********

上节分析了tp5的自动加载部分，根据代码执行顺序，base.php的59-64行

.. code-block:: php

    // 注册错误和异常处理机制
    thinkError::register();

    // 加载惯例配置文件
    thinkConfig::set(include THINK_PATH . 'convention' . EXT);



下面的加载配置文件不用说，现在重点看一下异常处理。

打开 ``library/think/Error.php`` ， ``register`` 函数。

.. code-block:: php

    public static function register()
    {
        error_reporting(E_ALL);
        // 通过 set_error_handler() 函数设置用户自定义的错误处理程序，然后触发错误（通过 trigger_error()）
        set_error_handler([__CLASS__, 'appError']);
        //设置默认的异常处理程序，用于没有用 try/catch 块来捕获的异常。 在 exception_handler 调用后异常会中止。
        set_exception_handler([__CLASS__, 'appException']);
        /*
         * register_shutdown_function 的函数,可以让我们设置一个当执行关闭时可以被调用的另一个函数。
         * 也就是说当我们的脚本执行完成或意外死掉导致PHP执行即将关闭时,我们的这个函数将会被调用。
         * 可以这样理解调用条件：
         * 1、当页面被用户强制停止时
         * 2、当程序代码运行超时时
         * 3、当PHP代码执行完成时，代码执行存在异常和错误、警告
         */
        register_shutdown_function([__CLASS__, 'appShutdown']);
    }

通过 ``error_reporting()`` 来指定php的报错级别。 ``E_ALL`` 为显示所有报错信息，所以在运行时 ``NOTICE`` 级别的警告也会显示，如果不想显示 ``NOTICE`` 信息，可以在项目 ``common`` 文件中重新设置一下报错级别： ``error_reporting(E_ALL ^ E_NOTICE);``

- set_error_handler:指定appError来处理系统异常
- set_exception_handler:来指定appException来处理用户抛出的异常
- register_shutdown_function:来指定appShutdown处理超时异常

然后使用 ``getExceptionHandler`` 方法来获取异常处理对象的实例。

.. code-block:: php

    public static function getExceptionHandler()
    {
        static $handle;
        if (!$handle) {
            // 异常处理handle
            $class = Config::get('exception_handle');
            if ($class && class_exists($class) && is_subclass_of($class, "\\think\\exception\\Handle")) {
                $handle = new $class;
            } else { //使用默认处理器
                $handle = new Handle;
                if ($class instanceof \Closure) {
                    $handle->setRender($class);
                }
            }
        }
        return $handle;
    }

里面有一句 ``$class = Config::get('exception_handle');`` 也就是说我们 **可以通过修改配置参数(exception_handle)来指定新的异常处理对象。**

``library/think/exception/`` 下是几个异常处理类，主要就是在开启 ``debug`` 的情况下输出异常等级，异常信息，异常代码等等。然后也对 ``CLI`` 模式下做了异常输出的处理。

