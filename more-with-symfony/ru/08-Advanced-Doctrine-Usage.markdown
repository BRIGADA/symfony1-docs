Продвинутое использование Doctrine
==================================

*By Jonathan H. Wage*

Написание поведения Doctrine
----------------------------

В этом разделе мы покажем как вы можете опысывать свои собственные модели поведения в Doctrine 1.2.
Мы создадим пример, который упростит вам содержание в кеше количества взаимоотношений,
что позволит не запрашивать это количество каждый раз.

Функциональность предельно проста. Для всех требующих счётчика отношений,
поведение добавит столбец к модели, в котором будет сохраняться текущее значение.

### Схема

Вот схема, которую мы будем использовать для начала. Позже мы изменим её и добавим определение
`actAs` для поведения, которое напишем:

    [yml]
    # config/doctrine/schema.yml
    Thread:
      columns:
        title:
          type: string(255)
          notnull: true

    Post:
      columns:
        thread_id:
          type: integer
          notnull: true
        body:
          type: clob
          notnull: true
      relations:
        Thread:
          onDelete: CASCADE
          foreignAlias: Posts

Теперь построем все необходимые классы для этой схемы:

    $ php symfony doctrine:build --all

### Шаблон

В начале нам необходимо написать простой дочерний класс `Doctrine_Template`, который
будет отвечать за добавление хранящих счётчики столбцов к модели.

Вы можете просто поместить этот код куда-нибудь в каталоги `lib/` проекта и symfony
автоматически загрузит его для вас:

    [php]
    // lib/count_cache/CountCache.class.php
    class CountCache extends Doctrine_Template
    {
      public function setTableDefinition()
      {
      }

      public function setUp()
      {
      }
    }

Теперь изменим модель `Post`, указав в `actAs` поведение `CountCache`:

    [yml]
    # config/doctrine/schema.yml
    Post:
      actAs:
        CountCache: ~
      # ...

Теперь, когда наша модель `Post` использует поведение `CountCache` позвольте рассказать
немного о том, что собственно происходит.

При обработке информации о модели вызываются методы
`setTableDefinition()` и `setUp()` всех прикреплённых к ней поведений. Это же произойдёт
и с классом `BasePost` в `lib/model/doctrine/base/BasePost.class.php`. Таким образом вы можете добавить
некоторые вещи к модели прямо на лету. Это могут быть столбцы, отношения, обработчики событий и т.д.

Теперь, когда вы немного понимаете происходящее, давайте наделим поведение `CountCache`
чем нибудь полезным:

    [php]
    class CountCache extends Doctrine_Template
    {
      protected $_options = array(
        'relations' => array()
      );

      public function setTableDefinition()
      {
        foreach ($this->_options['relations'] as $relation => $options)
        {
          // Создаём имя столбца, если оно не задано
          if (!isset($options['columnName']))
          {
            $this->_options['relations'][$relation]['columnName'] = 'num_'.Doctrine_Inflector::tableize($relation);
          }

          // Добавление столбца к связанной модели
          $columnName = $this->_options['relations'][$relation]['columnName'];
          $relatedTable = $this->_table->getRelation($relation)->getTable();
          $this->_options['relations'][$relation]['className'] = $relatedTable->getOption('name');
          $relatedTable->setColumn($columnName, 'integer', null, array('default' => 0));
        }
      }
    }

Код выше добавляет столбец, который будет обслуживать счётчик для связанной модели.
Таким образом в нашем случае, мы добавляем поведение к модели `Post` для взаимосвязи `Thread`.
Мы хотим хранить количество сообщений любого экземпляра `Thread` в столбце с именем `num_posts`.
Поэтому изменим YAML-схему, включив в определение поведения дополнительные опции:

    [yml]
    # ...

    Post:
      actAs:
        CountCache:
          relations:
            Thread:
              columnName: num_posts
              foreignAlias: Posts
      # ...

Теперь модель `Thread` содержит столбец `num_posts`, который сохраняет актуальное значение счётчика
сообщений каждого треда.

### Обработчик событий

Следующим шагом построения поведения будет написание обработчика собитий, 
который будет отвечать за поддержание актуальности счётчика при вставке новых записей,
удалении одной или нескольких записей:

    [php]
    class CountCache extends Doctrine_Template
    {
      // ...

      public function setTableDefinition()
      {
        // ...

        $this->addListener(new CountCacheListener($this->_options));
      }
    }

Перед тем как двинуться дальше, нам необходимо определить класс `CountCacheListener` наследующий `Doctrine_Record_Listener`.
Он принимает массив опций, которые просто передаются из в обработчик из шаблона:

    [php]
    // lib/model/count_cache/CountCacheListener.class.php

    class CountCacheListener extends Doctrine_Record_Listener
    {
      protected $_options;

      public function __construct(array $options)
      {
        $this->_options = $options;
      }
    }

Теперь для поддержания актуальности счётчика мы должны задействовать следующие события:

 * **postInsert()**: Увеличивает счётчик при вставке нового объекта;

 * **postDelete()**: Уменьшает счётчик при удалении объекта;

 * **preDqlDelete()**: Уменьшает счётчик при удалении записей через DQL.

Вначале определим метод `postInsert()`:

    [php]
    class CountCacheListener extends Doctrine_Record_Listener
    {
      // ...

      public function postInsert(Doctrine_Event $event)
      {
        $invoker = $event->getInvoker();
        foreach ($this->_options['relations'] as $relation => $options)
        {
          $table = Doctrine::getTable($options['className']);
          $relation = $table->getRelation($options['foreignAlias']);

          $table
            ->createQuery()
            ->update()
            ->set($options['columnName'], $options['columnName'].' + 1')
            ->where($relation['local'].' = ?', $invoker->$relation['foreign'])
            ->execute();
        }
      }
    }

Этот код увеличивает счётчики на единицу для всех сконфигурированных взаимосвязей
через вызов DQL-запроса UPDATE при вставке нового объекта:

    [php]
    $post = new Post();
    $post->thread_id = 1;
    $post->body = 'body of the post';
    $post->save();

Для `Thread` с `id` равным `1` увеличивается значение столбца `num_posts` на `1`.

Теперь когда счётчики увеличиваются при вставке новых объектов, 
нам необходимо обработать удаление объектов и соответствующее уменьшение счётчиков.
Мы сделаем это, реализовав метод `postDelete()`:

    [php]
    class CountCacheListener extends Doctrine_Record_Listener
    {
      // ...

      public function postDelete(Doctrine_Event $event)
      {
        $invoker = $event->getInvoker();
        foreach ($this->_options['relations'] as $relation => $options)
        {
          $table = Doctrine::getTable($options['className']);
          $relation = $table->getRelation($options['foreignAlias']);

          $table
            ->createQuery()
            ->update()
            ->set($options['columnName'], $options['columnName'].' - 1')
            ->where($relation['local'].' = ?', $invoker->$relation['foreign'])
            ->execute();
        }
      }
    }

Метод `postDelete()` очень похож на `postInsert()` за исключением того, что значение
столбца `num_posts` уменьшается на `1`, а не увеличивается.
Он обработает следующий, если мы удалим запись `$post`, которую создали ранее:

    [php]
    $post->delete();

Последним элементом головоломки станет обработка удаления записей через DQL.
Мы решим это при помощи метода `preDqlDelete()`:

    [php]
    class CountCacheListener extends Doctrine_Record_Listener
    {
      // ...

      public function preDqlDelete(Doctrine_Event $event)
      {
        foreach ($this->_options['relations'] as $relation => $options)
        {
          $table = Doctrine::getTable($options['className']);
          $relation = $table->getRelation($options['foreignAlias']);

          $q = clone $event->getQuery();
          $q->select($relation['foreign']);
          $ids = $q->execute(array(), Doctrine::HYDRATE_NONE);

          foreach ($ids as $id)
          {
            $id = $id[0];

            $table
              ->createQuery()
              ->update()
              ->set($options['columnName'], $options['columnName'].' - 1')
              ->where($relation['local'].' = ?', $id)
              ->execute();
          }
        }
      }
    }

Этот код клонирует запрос `DQL DELETE` и трансформирует его в `SELECT`, который позволяет получить `ID` удаляемых записей,
а затем использует их для обновления счётчиков.

Имея подобный сценарий, мы автоматизируем уменьшение счётчиков при выполнении следующего действия:

    [php]
    Doctrine::getTable('Post')
      ->createQuery()
      ->delete()
      ->where('id = ?', 1)
      ->execute();

Счётчики уменьшаются даже если мы удаляем несколько записей:

    [php]
    Doctrine::getTable('Post')
      ->createQuery()
      ->delete()
      ->where('body LIKE ?', '%cool%')
      ->execute();

>**NOTE**
>Для того, чтобы метод `preDqlDelete()` вызывался, вы должны включить специальный атрибут.
>По умолчанию, обратные вызовы DQL выключены из-за вызываемой ими дополнительно нагрузки.
>Поэтому, если вы хотите использовать данную возможность, вам необходимо принудительно включить её.
>
>     [php]
>     $manager->setAttribute(Doctrine_Core::ATTR_USE_DQL_CALLBACKS, true);

Вот и всё! Разработка нашего поведения завершена. Всё что нам осталось сделать -- это неписать для него небольшой тест.

### Тестирование

После реализации кода, давайте проверим его с помощью следующих начальных данных:

    [yml]
    # data/fixtures/data.yml

    Thread:
      thread1:
        title: Test Thread
        Posts:
          post1:
            body: This is the body of my test thread
          post2:
            body: This is really cool
          post3:
            body: Ya it is pretty cool

Перестроим все классы и загрузим данные:

    $ php symfony doctrine:build --all --and-load

Теперь давайте запустим тест, чтобы увидеть работу счётчиков:

    $ php symfony doctrine:dql "FROM Thread t, t.Posts p"
    doctrine - executing: "FROM Thread t, t.Posts p" ()
    doctrine -   id: '1'
    doctrine -   title: 'Test Thread'
    doctrine -   num_posts: '3'
    doctrine -   Posts:
    doctrine -     -
    doctrine -       id: '1'
    doctrine -       thread_id: '1'
    doctrine -       body: 'This is the body of my test thread'
    doctrine -     -
    doctrine -       id: '2'
    doctrine -       thread_id: '1'
    doctrine -       body: 'This is really cool'
    doctrine -     -
    doctrine -       id: '3'
    doctrine -       thread_id: '1'
    doctrine -       body: 'Ya it is pretty cool'

Вы видите, что в модели `Thread` есть столбец `num_posts`, значение которого равно трём.
Если мы удалим одно из сообщений следующим кодом, то счётчик должен измениться:

    [php]
    $post = Doctrine_Core::getTable('Post')->find(1);
    $post->delete();

Вы видите, что запись действительно удалена и значение счётчика обновлено:

    $ php symfony doctrine:dql "FROM Thread t, t.Posts p"
    doctrine - executing: "FROM Thread t, t.Posts p" ()
    doctrine -   id: '1'
    doctrine -   title: 'Test Thread'
    doctrine -   num_posts: '2'
    doctrine -   Posts:
    doctrine -     -
    doctrine -       id: '2'
    doctrine -       thread_id: '1'
    doctrine -       body: 'This is really cool'
    doctrine -     -
    doctrine -       id: '3'
    doctrine -       thread_id: '1'
    doctrine -       body: 'Ya it is pretty cool'

Это работает даже если вы удалите оставшиеся две записи:

    [php]
    Doctrine_Core::getTable('Post')
      ->createQuery()
      ->delete()
      ->where('body LIKE ?', '%cool%')
      ->execute();

Мы удалили все связанные записи, и в `num_posts` должен быть ноль:

    $ php symfony doctrine:dql "FROM Thread t, t.Posts p"
    doctrine - executing: "FROM Thread t, t.Posts p" ()
    doctrine -   id: '1'
    doctrine -   title: 'Test Thread'
    doctrine -   num_posts: '0'
    doctrine -   Posts: {  }

Вот и всё! Я надеюсь, эта статья будет полезна не только из-за полученных знаний,
но и само разработанное поведение может быть вам полезно!

Использование кеширования результатов Doctrine
----------------------------------------------

Для высоконагруженных веб-приложений требуется кешировать информацию для снижения нагрузки на процессорные ресурсы.
В Doctrine 1.2 мы сделали множество улучшений кеширования результатов, дающих вам больший контроль над удалением записей из кеша.
Раньше нельзя было указывать ключ, под которым бы хранился элемент кеша, поэтому вы немогли никак идентифицировать его для удаления.

В этом разделе мы продемонстрируем вам пример использования кеширования результатов
для всех ваших связанных с пользователями запросов, а также использование событий для очистки кеша
при изменении некоторых данных.

### Схема

Для примера мы будем использовать следующую схему:

    [yml]
    # config/doctrine/schema.yml
    User:
      columns:
        username:
          type: string(255)
          notnull: true
          unique: true
        password:
          type: string(255)
          notnull: true

Теперь построим классы по этой схеме следующей командой:

    $ php symfony doctrine:build --all

В результате у вас появится класс `User`:

    [php]
    // lib/model/doctrine/User.class.php
    /**
     * User
     *
     * This class has been auto-generated by the Doctrine ORM Framework
     *
     * @package    ##PACKAGE##
     * @subpackage ##SUBPACKAGE##
     * @author     ##NAME## <##EMAIL##>
     * @version    SVN: $Id: Builder.php 6508 2009-10-14 06:28:49Z jwage $
     */
    class User extends BaseUser
    {
    }

Далее в этой статье вам потребуется добавлять в него некоторый код.

### Конфигурирование кеша результатов

In order to use the result cache we need to configure a cache driver for the
queries to use. This can be done by setting the `ATTR_RESULT_CACHE` attribute.
We will use the APC cache driver as it is the best choice for production. If you
do not have APC available, you can use the `Doctrine_Cache_Db` or `Doctrine_Cache_Array`
driver for testing purposes.

We can set this attribute in our `ProjectConfiguration` class. Define a `configureDoctrine()` method:

    [php]
    // config/ProjectConfiguration.class.php

    // ...
    class ProjectConfiguration extends sfProjectConfiguration
    {
      // ...

      public function configureDoctrine(Doctrine_Manager $manager)
      {
        $manager->setAttribute(Doctrine_Core::ATTR_RESULT_CACHE, new Doctrine_Cache_Apc());
      }
    }

Now that we have the result cache driver configured, we can begin to actually
use this driver to cache the result sets of the queries.

### Sample Queries

Представьте, что в вашем приложении есть несколько связанных 
Now imagine in your application you have a bunch of user related queries
and you want to clear them whenever some user data is changed.

Вот простой запрос, который позволяет получить отсортированный в алфовитном порядке список пользователей:

    [php]
    $q = Doctrine_Core::getTable('User')
        ->createQuery('u')
        ->orderBy('u.username ASC');

Вы можете включить кеширование для этого запроса, используя метод `useResultCache()`:

    [php]
    $q->useResultCache(true, 3600, 'users_index');

>**NOTE**
>Обратите внимание на третий аргумент.
>Это ключ, который будет использоваться для сохранения результатов в кеше.
>Он упрощает идентификацию запроса и удаление его из кеша.

Теперь когда мы выполним запрос, он обратится к базе данных за результатами
и сохранит их в кеше под ключом `users_index`, а последующие запросы будут получать
результаты уже из кеша минуя обращение к базе данных:

    [php]
    $users = $q->execute();

>**NOTE**
>Not only does this save processing on your database server, it also bypasses
>the entire hydration process as Doctrine stores the hydrated data. This means
>that it will relieve some processing on your web server as well.

Now if we check in the cache driver, you will see that there is an entry named
`users_index`:

    [php]
    if ($cacheDriver->contains('users_index'))
    {
      echo 'cache exists';
    }
    else
    {
      echo 'cache does not exist';
    }

### Удаление кеша

Now that the query is cached, we need to learn a bit about how we can delete
that cache. We can delete it manually using the cache driver API or we can utilize
some events to automatically clear the cache entry when a user is inserted or modified.

#### Cache Driver API

First we will just demonstrate the raw API of the cache driver before we implement
it in an event.

>**TIP**
>Для доступа к экземпляру кеша результатов, вы можете воспользоваться экземпляром класса `Doctrine_Manager`.
>
>     [php]
>     $cacheDriver = $manager->getAttribute(Doctrine_Core::ATTR_RESULT_CACHE);
>
>Если у вас ешё нет долтупа к переменной `$manager`, вы можете полутить её экземпляр следующим кодом.
>
>     [php]
>     $manager = Doctrine_Manager::getInstance();

Теперь мы можем использовать API для удаления записей из кеша:

    [php]
    $cacheDriver->delete('users_index');

Вполне вероятно, что у вас будет более одного начинающегося с `users_` запроса,
и вам захочется удалить все их кешированные результаты.
В этом случае, метод `delete()` нам не поможет. Однако поможет метод `deleteByPrefix()`,
который позволяет нам удалять все элементы кеша, содержащие указанный префикс.
Пример:

    [php]
    $cacheDriver->deleteByPrefix('users_');

Также есть несколько других методов для удаления записей из кеша, помимо `deleteByPrefix()`:

 * `deleteBySuffix($suffix)`: Удаляет элементы кеша, содержащие переданный суффикс;

 * `deleteByRegexp($regex)`: Удаляет элементы кеша, которые соответствуют переданному регулярному выражению;

 * `deleteAll()`: Удаляет все элементы кеша.

### Удаление по событиям

The ideal way to clear the cache would be to automatically clear it whenever some
user data is modified. We can do this by implementing a `postSave()` event in our
`User` model class definition.

Remember the `User` class we talked about earlier? Now we need to add some code to it
so open the class in your favorite editor and add the following `postSave()` method:

    [php]
    // lib/model/doctrine/User.class.php

    class User extends BaseUser
    {
      // ...

      public function postSave($event)
      {
        $cacheDriver = $this->getTable()->getAttribute(Doctrine_Core::ATTR_RESULT_CACHE);
        $cacheDriver->deleteByPrefix('users_');
      }
    }

Now if we were to update a user or insert a new user it would clear the cache for
all the user related queries:

    [php]
    $user = new User();
    $user->username = 'jwage';
    $user->password = 'changeme';
    $user->save();

The next time the queries are invoked it will see the cache does not exist and fetch
the data fresh from the database and cache it again for subsequent requests.

While this example is very simple it should demonstrate pretty well how you can
use these features to implement fine grained caching on your Doctrine queries.

Написание гидратора Doctrine
----------------------------

One of the key features of Doctrine is the ability to transform a `Doctrine_Query`
object to the various result set structures. This is the job of the Doctrine
hydrators and up until Doctrine 1.2, the hydrators were all hardcoded and not
open for developers to write custom ones. Now that this has changed it is possible
to write a custom hydrator and create whatever data structure that is desired
from the data the database gives you back when executing a `Doctrine_Query` instance.

In this example we will build a hydrator that will be extremely simple and easy to
understand, yet very useful. It will allow you to select two columns and hydrate
the data to a flat array where the first selected column is the key and the second
select column is the value.

### Схема и начальные данные

To get started first we need a simple schema to run our tests with. We will just use
a simple `User` model:

    [yml]
    # config/doctrine/schema.yml
    User:
      columns:
        username: string(255)
        is_active: string(255)

We will also need some data fixtures to test against, so copy the fixtures from below:

    [yml]
    # data/fixtures/data.yml
    User:
      user1:
        username: jwage
        password: changeme
        is_active: 1
      user2:
        username: jonwage
        password: changeme
        is_active: 0

Now build everything with the following command:

    $ php symfony doctrine:build --all --and-load

### Написание гидратора

To write a hydrator all we need to do is write a new class which extends `Doctrine_Hydrator_Abstract`
and must implement a `hydrateResultSet($stmt)` method. It receives the `PDOStatement`
instance used to execute the query. We can then use that statement to get the raw
results of the query from PDO then transform that to our own structure.

Let's create a new class named `KeyValuePairHydrator` and place it in the `lib/`
directory so that symfony can autoload it:

    [php]
    // lib/KeyValuePairHydrator.class.php
    class KeyValuePairHydrator extends Doctrine_Hydrator_Abstract
    {
      public function hydrateResultSet($stmt)
      {
        return $stmt->fetchAll(Doctrine_Core::FETCH_NUM);
      }
    }

The above code as it is now would just return the data exactly as it is from PDO.
This isn't quite what we want. We want to transform that data to our own key => value
pair structure. So let's modify the `hydrateResultSet()` method a little bit to do
what we want:

    [php]
    // lib/KeyValuePairHydrator.class.php
    class KeyValuePairHydrator extends Doctrine_Hydrator_Abstract
    {
      public function hydrateResultSet($stmt)
      {
        $results = $stmt->fetchAll(Doctrine_Core::FETCH_NUM);
        $array = array();
        foreach ($results as $result)
        {
          $array[$result[0]] = $result[1];
        }

        return $array;
      }
    }

Well that was easy! The hydrator code is finished and it does exactly what we want
so let's test it!

### Использование гидратора

To use and test the hydrator we first need to register it with Doctrine so that
when we execute some queries, Doctrine is aware of the hydrator class we have written.

To do this, register it on the `Doctrine_Manager` instance in the `ProjectConfiguration`:

    [php]
    // config/ProjectConfiguration.class.php

    // ...
    class ProjectConfiguration extends sfProjectConfiguration
    {
      // ...

      public function configureDoctrine(Doctrine_Manager $manager)
      {
        $manager->registerHydrator('key_value_pair', 'KeyValuePairHydrator');
      }
    }

Now that we have the hydrator registered, we can make use of it with `Doctrine_Query`
instances. Here is an example:

    [php]
    $q = Doctrine_Core::getTable('User')
      ->createQuery('u')
      ->select('u.username, u.is_active');

    $results = $q->execute(array(), 'key_value_pair');
    print_r($results);

Executing the above query with the data fixtures we defined above would result
in the following:

    Array
    (
        [jwage] => 1
        [jonwage] => 0
    )

Well that is it! Pretty simple huh? I hope this will be useful for you and as a result
the community gets some awesome new hydrators contributed.
