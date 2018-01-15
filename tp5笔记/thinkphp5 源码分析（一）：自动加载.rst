******
类的自动加载
******

入口
====
作为单入口框架，就从入口文件看起，按照tp5文档所示的规范，入口文件应该是放在public/ 下。

那么为什么大多数要把入口放到子文件夹下呢？这是一个小技巧。

- 第一为了 **动静分离** ，因为现在的php框架一般都是单入口，既然是单入口，那么必然要做rewrite，如果把静态文件和程序文件放到一起。框架路由势必要对每一个请求进行筛选。所以这些框架不约而同的把资源文件和程序文件区分开来，放在了不同的文件夹下。从整体来看，也就是为什么入口会在子目录下了。

第二是为了 **安全** ，linux下的权限划分非常严格，分别分为读、写、执行，在这个基础上又分为文件所有组、所在组、其他组。这样划分可以更好的对文件权限进行梳理，避免上传漏洞（用户上传php文件被执行）等等。

打开public/index.php

.. code-block:: php

    define('APP_PATH', __DIR__ . '/../application/');
    // 加载框架引导文件
    require __DIR__ . '/../thinkphp/start.php';

只有两行代码，定义 ``APP_PATH`` ，加载 ``/../thinkphp/start.php`` 。APP_PATH 可以自己修改。然后打开 ``/../thinkphp/start.php``

.. code-block:: php

    namespace think;
    // ThinkPHP 引导文件
    // 加载基础文件
    require __DIR__ . '/base.php';
    // 执行应用
    App::run()->send();

也只有三行代码，定义命名空间，加载基础文件，启动应用。这里注意一下命名空间，所有thinkphp类都在think及其子命名空间下。程序中用到框架类的时候要先use 该类的命名空间；

环境配置
========
然后我们打开 ``base.php`` 12-31行定义了一坨常量。注意里面 ``defined('THINK_PATH') or define('THINK_PATH', __DIR__ . DS);`` 这种定义方式，先判断时候存在，如果不存在则定义。也就是说我们可以在这行代码之前（一般在index.php中）执行定义这个常量，而不会被覆盖。

.. code-block:: php

    // 载入Loader类
    require CORE_PATH . 'Loader.php';

载入Loader类，这个类比较重要，实现了类的自动加载。

.. code-block:: php

    // 加载环境变量配置文件
    if (is_file(ROOT_PATH . 'env' . EXT)) {
        $env = include ROOT_PATH . 'env' . EXT;
        foreach ($env as $key => $val) {
            $name = ENV_PREFIX . strtoupper($key);
            if (is_bool($val)) {
                $val = $val ? 1 : 0;
            } elseif (!is_scalar($val)) {
                continue;
            }
            putenv("$name=$val");
        }
    }

加载环境变量配置文件，可能很多人不理解是干什么用的。

我们假设一个场景，你在公司和家里开发程序，在内网服务器上进行测试，在外网服务器上部署，所有的配置不能可能全部相同（比如数据库帐号密码、文件路径等等）。总不能每次都改配置吧？如果做负载、有几十个服务器怎么部署？总不能都用ftp上传，然后改配置吧？

所以现在主流的做法就是区分环境（开发环境、测试环境、生产环境），然后程序自动加载不同的配置。但是通过什么区分呢？方法有很多，但是大多数都是选择通过环境变量来区分，然后加载对应的配置文件。然后使用 ``git`` 做版本控制，然后在服务器部署同步脚本，通过 ``git push`` 钩子进行代码同步，以达到自动化部署的模式。当然也还有其他方式，但是大多都类似。

类的自动加载
============
为什么要使用自动加载呢？因为像java、C等编译型语言在编译过程中会把程序中引用的库、包等等自动引入进来。但是php是脚本行语言啊，没有编译过程，怎么办呢？最早期的程序都是手动引入，比如早期的 ``xxshop`` 、 ``xxcms`` ，都是写一坨 ``require`` 、 ``include`` 。又搓又不方便，对于世界上最好的语言来说这样多丢面啊，所以我们需要用自动加载让我们最好的语言看起来更有B格（至于某些性能论的同学会说自动加载影响性能啊之类的，请用汇编！）。

我们继续看 ``base.php`` 的54行 ``\think\Loader::register();`` 注册类的自动加载，从这一行之后就可以使用符合自动加载规范的任何类了。

比如56-60行，虽然没有加载对应的文件，但是通过自动加载就可以直接使用。

.. code-block:: php

    // 注册错误和异常处理机制
    \think\Error::register();
    // 加载惯例配置文件
    \think\Config::set(include THINK_PATH . 'convention' . EXT);

接下来我们看一下自动加载的实现方法。打开 ``Loader.php`` ，按照上面的执行顺序，先看 ``Loader`` 类的 ``register`` 方法。

核心是：

.. code-block:: php

    // 注册自动加载机制
    public static function register($autoload = '')
    {
        // 注册系统自动加载
        spl_autoload_register($autoload ?: 'think\\Loader::autoload', true, true);
        // 注册命名空间定义
        self::addNamespace([
            'think'    => LIB_PATH . 'think' . DS,
            'behavior' => LIB_PATH . 'behavior' . DS,
            'traits'   => LIB_PATH . 'traits' . DS,
        ]);
        // 加载类库映射文件
        if (is_file(RUNTIME_PATH . 'classmap' . EXT)) {
            self::addClassMap(__include_file(RUNTIME_PATH . 'classmap' . EXT));
        }

        // Composer自动加载支持
        if (is_dir(VENDOR_PATH . 'composer')) {
            self::registerComposerLoader();
        }

        // 自动加载extend目录
        self::$fallbackDirsPsr4[] = rtrim(EXTEND_PATH, DS);
    }

``spl_autoload_register`` 方法可能很多人都有了解，在我们实例化一个当前已加载文件中不存在的类后（比如在 ``a.php`` 中 ``new`` 一个类，会先在 ``a.php`` 和已加载的文件中找），会执行此方法指定的函数，并把类名传递进去。在这个函数中如果能正确加载到该文件，那么也可以实例化成功，并不会报错。所以借助此函数可以达到自动加载。

按照命名空间映射方式加载类
--------------------------
首先我们知道当 ``new`` 一个不存在的类时，如果使用 ``spl_autoload_register`` 定义了一个处理函数，那么这个函数可以获得一个参数，参数名是 ``new``  的类名。比如从前面 ``base.php`` 中我们看到 ``\think\Error::register();`` 使用think命名空间下的 ``Error`` 类的 ``register`` 静态方法，但是我们并没有引入这个文件。可是我们可以在 ``spl_autoload_register`` 注册的函数中得到一个参数 ``think\Error`` ，如果我们的命名空间按照文件夹格式的方法命名（这也是推荐的、常用的命名方式），那么就可以通过该参数来加载对应的文件。

按照自定义的映射来加载类
-------------------------
上面解决了命名空间类自动加载，但是如果特殊情况下没有按照文件夹的格式来进行命名空间的命名，那么就需要手动指定映射关系。 ``self::addClassMap(__include_file(RUNTIME_PATH . 'classmap' . EXT));`` 就是来定义手动指定文件与文件路径映射关系的。

扩展插件类自动加载
------------------
一个框架是否好用，很大程度取决于它的扩展能力。所以自动加载除了要处理自身类库的加载、还要处理扩展类库的自动加载。tp5支持使用两种方式来扩展类库一种是 ``Composer`` ，一种是手动放入 ``extend`` 目录。

``Composer`` 不用多说，就像npm之于nodejs、yum之于Centos、apt-get之于、Ubuntu。一个php的包管理工具。 ``Composer`` 有一套自己的自动加载机制，tp5这里只不过是调用了 ``Composer`` 自己的注册自动加载函数的方法。有兴趣的同学可以看一下 ``registerComposerLoader`` 方法，以及 ``vendor/composer`` 下几个 ``autoload`` 开头的文件。原理基本上和上面的一致。


加载了映射文件。然后我们看 ``spl_autoload_register`` 中指定的函数： ``autoload`` 。
这个不用详细解释了，先处理由 ``addNamespace`` 设定的命名空间别名，然后通过 ``findFile`` 来处理映射关系，得到真实的路径，并加载文件。

而 ``__autoload()`` 函数具有类似的功能。但是为什么用的很少呢？因为 ``__autoload()`` 只能指定一个函数，而 ``spl_autoload_register`` 可以注册多个函数来处理这个逻辑。一旦业务复杂 ``__autoload()`` 就完全不能胜任。

