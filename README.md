# symfony-sample
十分钟学习Symfony
Symfony是一个强大的基于PHP的Web开发框架,在这里我们用十分钟的时间来做一个简单的增删改查的程序， 任何不熟悉Symfony的人都可以通过这个教程完成自己的第一个Symfony程序。

如果需要这个样例程序的全部源代码，可以访问 这里，或者通过下面的方式获取源代码：
	
$git clone https://github.com/saharabear/symfony-sample.git
(这是原作者的仓库如今已经清理，更新本仓库代码即可；以下教程也为原作者整理；)


项目初始化

首先，需要你在自己的电脑中安装PHP环境并安装git.这方面的内容属于基础内容，网络上有大量的教程，在这里就不多介绍了，不过要提示的一点是：PHP从5.4开始， 已经内置了测试用服务器，Symfony也拥抱了这个由PHP内置的服务器，只需要在命令行中使用$php app/console server:run 就可以 启动基于Symfony框架的PHP程序进行测试，因此不必要使用XAMPP这一类复杂的集成环境，直接安装PHP并保证在命令行下可以执行php命令就可以了。

然后，我们需要建立一个新的目录，名字叫symfony-sample，Symfony使用一个叫composer的程序管理各种类库的依赖关系，因此如果你的机器上 安装了composer,就可以直接跳过这一步，如果没有安装，可以用下面的命令安装最新版本的composer.

$cd symfony-sample
$curl -sS https://getcomposer.org/installer | php

如果希望了解更多关于composer的信息，可以参考这个网站。

安装完成composer后，我们可以开始安装当前最新版本的Symfony2.6.0

$php composer.phar create-project symfony/framework-standard-edition mysampleproject/ 2.6.0

安装过程中，需要填写数据库等信息，在这个例子中，我们会使用mysql数据库，因此你可以一路按回车键，先不要关心这一切配置应该如何填写。反正 Symfony会在安装成功后，生成一个配置文件，叫app/config/parameters.yml,下面我会提供一个parameters.yml文件的 内容样本，只要复制进去就可以了，先不必关注这么多细节。

刚才创建mysampleproject以后，在symfony-sample目录下生成了mysampleproject目录，我习惯于将程序放在项目的根目录下，因此执行下面的几个命令， 就可以把项目从symfony-sample/mysampleproject目录中，移到symfony-sample目录中

	
$mv mysampleproject/* ./
$rm -rf mysampleproject

理论上来讲，我们已经完成了Symfony项目的创建，不过刚才提到的parameters.yml文件还没有解释。这个parameters.yml是Symfony的全局配置文件， 无论是数据库配置信息还是其他的各种配置，都可以放在这个文件中。下面是我们需要使用的测试用的parameters.yml,记得把最后一行的值修改为一个随机值
	
# This file is auto-generated during the composer install
parameters:
    database_driver: pdo_mysql
    database_host: localhost
    database_port: 3306
    database_name: symfony
    database_user: root
    database_password: root
    mailer_transport: smtp
    mailer_host: localhost
    mailer_user: null
    mailer_password: null
    locale: en
    secret: ChangeThisLineAsYouWish_ioiuqwoieru

直接用这段，替换掉app/config/parameters.yml文件中的内容，然后编辑app/config/config.yml,找到下面几行，把最后一行添加进去并保存。
	
driver:   "%database_driver%"
host:     "%database_host%"
port:     "%database_port%"
dbname:   "%database_name%"
user:     "%database_user%"
password: "%database_password%"
charset:  UTF8
path:     "%database_path%"

好了，这样我们就完成了基本的Symfony程序的配置，你现在有了一个配置好了数据库，邮件发送器，日志系统的基本程序原型。下面，我们就开始编写自己的Symfony程序。
建立Bundle

先说一下什么是Bundle。Symfony是以DI为核心的，可能你不知道什么是DI，没关系，这不重要，你可以把Symfony的DI理解成为一个功能池，把程序中的所有功能都做成Bundle，或者你把Bundle理解成一组php文件组合而成的程序就可以。 比如用户注册，登录功能做成一个Bundle,你也可以把一个论坛的发帖回贴功能做成一个Bundle，自然也可以把文章管理做成一个Bundle，然后用一个Bundle去调用和配置不同的Bundle，那么你就可以把网站组装起来了，而你写的各种Bundle，在其他的应用程序中还可以继续复用，这样写的Bundle越多，可复用性就越强，制作新项目的时候也越有利。

我们现在就来建立自己的Bundle.在命令行中，使用命令：
	
$php app/console generate:bundle
Bundle namespace: Symfony/Bundle/SampleBundle
Bundle name [SymfonySampleBundle]:
Target directory [/home/saharabear/workspace/symfony-sample/src]:
Configuration format (yml, xml, php, or annotation): yml
Do you want to generate the whole directory structure [no]? yes
Do you confirm generation [yes]? yes
Generating the bundle code: OK
Checking that the bundle is autoloaded: OK
Confirm automatic update of your Kernel [yes]? yes
Enabling the bundle inside the Kernel: OK
Confirm automatic update of the Routing [yes]? yes

这样就成功建立了我们的Bundle，名字叫SymfonySampleBundle，我们使用的Bundle namespace是Symfony/Bundle/SampleBundle，这是一种约定，我们还可以建立其他的Bundle，比如Symfony/Bundle/PostBundle, 或者Symfony/Bundle/ArticleBundle，而对应的Bundle name就分别是SymfonyPostBundle或者SymfonyArticleBundle。你也可以自己建立这几个Bundle，这并不会影响当前我们的教程。

对了，在我们建立的Bundle中，分别会生成下面几个目录：

    Entity:这个目录并不是必须的，很多情况下只有在生成实体的时候才会生成，放置模型，也就是MVC中的M
    Controller:这个目录会生成DefaultController.php，你可以在这里建立自己的Controller控制器，也就是MVC中的C
    Resources:这个目录下面还有子目录，其中views放置的是模板，也就是MVC中的V，而public放置的是静态文件，比如js, css, images等等
    Tests:放置单元测试与集成测试的代码，在这个样例程序中暂时不需要
    DependencyInjection:与DI相关的目录，暂时也不需要去了解
    SymfonySampleBundle.php:当前这个Bundle的定义文件

更多细节可以去阅读Symfony的官方文档，而当前的重点是把这个Symfony的样例程序运行起来。
设计实体

在MVC的设计理念中，M是最重要的，因为M表达的内容是业务逻辑。我觉得如果这个地方往深入去探讨，会一直探讨到富血模型或者贫血模型，不过目前在这个教程中根本 不需要考虑这么多，你只需要知道实体就是MVC中的M，用于表达业务逻辑。比如说，我们要开发一个文章管理的系统，那么文章本身就代表的业务逻辑。比如，我们的文章要有 标题，内容，作者，那么这三项就属于业务逻辑，而标题不能够为空，不能超过200长度，内容不能为空，作者却是可以为空的，这些也属于业务逻辑。同时，这个文章需要被 存储起来，比如存储到数据库中，那么这个M就应该能够映射到数据库的表中。我们把这个M，叫实体。

还是少说废话，直接上代码。那么如何建立实体呢？当然不是从头一点一点地写，而是直接用下面的命令生成：

	
$php app/console generate:doctrine:entity
 
Welcome to the Doctrine2 entity generator
 
This command helps you generate Doctrine2 entities.
 
First, you need to give the entity name you want to generate.
You must use the shortcut notation like AcmeBlogBundle:Post.
The Entity shortcut name: SymfonySampleBundle:Article
 
Determine the format to use for the mapping information.
 
Configuration format (yml, xml, php, or annotation) [annotation]:yml
Instead of starting with a blank entity, you can add some fields now.
Note that the primary key will be added automatically (named id).
 
Available types: array, simple_array, json_array, object,
boolean, integer, smallint, bigint, string, text, datetime, datetimetz,
date, time, decimal, float, blob, guid.
 
New field name (press  to stop adding fields): title
Field type [string]:
Field length [255]: 200
 
New field name (press  to stop adding fields): content
Field type [string]: text
 
New field name (press  to stop adding fields): author
Field type [string]:
Field length [255]: 20
 
New field name (press  to stop adding fields):
 
Do you want to generate an empty repository class [no]? yes
 
Summary before generation
 
You are going to generate a "SymfonySampleBundle:Article" Doctrine2 entity
using the "yml" format.
 
Do you confirm generation [yes]? yes
 
Entity generation
 
Generating the entity code: OK
 
You can now start using the generated code!

经过这些命令，你会发现在Entity中建立了新的文件Article.php,代码如下：
	
namespace Symfony\Bundle\SampleBundle\Entity;
 
use Doctrine\ORM\Mapping as ORM;
 
/**
 * Article
 *
 * @ORM\Table()
 * @ORM\Entity(repositoryClass="Symfony\Bundle\SampleBundle\Entity\ArticleRepository")
 */
class Article
{
    /**
     * @var integer
     *
     * @ORM\Column(name="id", type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;
 
    /**
     * @var string
     *
     * @ORM\Column(name="title", type="string", length=200)
     */
    private $title;
 
    /**
     * @var string
     *
     * @ORM\Column(name="content", type="text")
     */
    private $content;
 
    /**
     * @var string
     *
     * @ORM\Column(name="author", type="string", length=20)
     */
    private $author;
 
    /**
     * Get id
     *
     * @return integer
     */
    public function getId()
    {
        return $this->id;
    }
 
    /**
     * Set title
     *
     * @param string $title
     * @return Article
     */
    public function setTitle($title)
    {
        $this->title = $title;
 
        return $this;
    }
 
    /**
     * Get title
     *
     * @return string
     */
    public function getTitle()
    {
        return $this->title;
    }
 
    /**
     * Set content
     *
     * @param string $content
     * @return Article
     */
    public function setContent($content)
    {
        $this->content = $content;
 
        return $this;
    }
 
    /**
     * Get content
     *
     * @return string
     */
    public function getContent()
    {
        return $this->content;
    }
 
    /**
     * Set author
     *
     * @param string $author
     * @return Article
     */
    public function setAuthor($author)
    {
        $this->author = $author;
 
        return $this;
    }
 
    /**
     * Get author
     *
     * @return string
     */
    public function getAuthor()
    {
        return $this->author;
    }
}

你可以一行不改地使用这些代码。这时候我们再来做几个神奇的操作：
	
$php app/console doctrine:schema:update --force</pre>

这个操作，已经帮助你通过Article.php建立了数据库和数据表，你不需要自己操作这个过程，下面我们还会对Article.php进行改造，而到时候只需要重新 执行上面的这个操作，Symfony会帮助你自动修改数据库的表结构。
添加约束

上面我们创建了Article.php，既然这个实体代表了具体的业务逻辑，因此我们要考虑几个现实的问题：

    用户必须填写标题和内容
    用户填写的标题不能超过200个字
    用户可以不填写作者

这些就属于业务逻辑，而我们可以修改Article.php如下，以增加相应的业务逻辑的约束：
	
namespace Symfony\Bundle\SampleBundle\Entity;
 
use Doctrine\ORM\Mapping as ORM;
use Symfony\Component\Validator\Constraints as Assert;
 
/**
 * Article
 *
 * @ORM\Table()
 * @ORM\Entity(repositoryClass="Symfony\Bundle\SampleBundle\Entity\ArticleRepository")
 */
class Article
{
    /**
     * @var integer
     *
     * @ORM\Column(name="id", type="integer")
     * @ORM\Id
     * @ORM\GeneratedValue(strategy="AUTO")
     */
    private $id;
 
    /**
     * @var string
     * @Assert\NotBlank(message="标题不可为空")
     * @Assert\Length(
     *     max=200,
     *     maxMessage="标题不能超过200个字"
     * )
     * @ORM\Column(name="title", type="string", length=200)
     */
    private $title;
 
    /**
     * @var string
     *
     * @Assert\NotBlank(message="文章内容不可为空")
     * @ORM\Column(name="content", type="text")
     */
    private $content;
 
    /**
     * @var string
     *
     * @ORM\Column(name="author", type="string", length=20,nullable=true)
     */
    private $author;
 
    /**
     * Get id
     *
     * @return integer
     */
    public function getId()
    {
        return $this->id;
    }
 
    /**
     * Set title
     *
     * @param string $title
     * @return Article
     */
    public function setTitle($title)
    {
        $this->title = $title;
 
        return $this;
    }
 
    /**
     * Get title
     *
     * @return string
     */
    public function getTitle()
    {
        return $this->title;
    }
 
    /**
     * Set content
     *
     * @param string $content
     * @return Article
     */
    public function setContent($content)
    {
        $this->content = $content;
 
        return $this;
    }
 
    /**
     * Get content
     *
     * @return string
     */
    public function getContent()
    {
        return $this->content;
    }
 
    /**
     * Set author
     *
     * @param string $author
     * @return Article
     */
    public function setAuthor($author)
    {
        $this->author = $author;
 
        return $this;
    }
 
    /**
     * Get author
     *
     * @return string
     */
    public function getAuthor()
    {
        return $this->author;
    }
}

然后执行同步数据库的操作：
	
$ php app/console doctrine:schema:update --force
Updating database schema...
Database schema updated successfully! "1" queries were executed
增删改查

好了，我们来做一个针对文章的增删改查操作。首先请执行下面的命令：
	
$ php app/console generate:doctrine:crud
 
  Welcome to the Doctrine2 CRUD generator
 
This command helps you generate CRUD controllers and templates.
 
First, you need to give the entity for which you want to generate a CRUD.
You can give an entity that does not exist yet and the wizard will help
you defining it.
 
You must use the shortcut notation like AcmeBlogBundle:Post.
 
The Entity shortcut name: SymfonySampleBundle:Article
 
By default, the generator creates two actions: list and show.
You can also ask it to generate "write" actions: new, update, and delete.
 
Do you want to generate the "write" actions [no]? yes
 
Determine the format to use for the generated CRUD.
 
Configuration format (yml, xml, php, or annotation) [annotation]: yml
Determine the routes prefix (all the routes will be "mounted" under this
prefix: /prefix/, /prefix/new, ...).
 
Routes prefix [/article]: /article
 
  Summary before generation
 
You are going to generate a CRUD controller for "SymfonySampleBundle:Article"
using the "yml" format.
 
Do you confirm generation [yes]? yes
 
  CRUD generation
 
Generating the CRUD code: OK
Generating the Form code: OK
 
  You can now start using the generated code!

然后请编辑DefaultController.php中的indexAction如下：
	
/**
 * @Route("/",name="welcome")
 * @Template()
 */
public function indexAction()
{
    return array();
}

编辑Resource/views/Default/index.html.twig内容如下：
	
<a href="{{path('article')}}">文章管理</a>

让我们看看神奇的事情，启动内置的测试服务器：
	
$php app/console server:run

好了，我们已经完成了这十分钟的博客，一切的代码都在Controller/ArticleController.php,Form/ArticleType.php,Resource/views/Article/*.html.twig中，我们已经完成了最基本的文章管理功能。当然在你熟悉Symfony以后，未必需要完全依靠Symfony帮你生成这些增删改查操作，可是起码Symfony用一个命令让一切都先运行起来了，这不就是我们所要的原型吗？
