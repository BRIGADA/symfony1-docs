Глава 1 - Создание форм
=========================

Форма состоит из полей различных типов (скрытых, текстовых, селекторов и отметок). Эта глава познакомит вас с созданием форми и управлением их полями в фреймворке Symfony.

Для выполнения примеров из этой книги вам потребуется как минимум Symfony 1.1. Также вам необходимо создать проект и приложение `frontend`. Пожалуйста, ознакомьтесь с введением для получения информации о создании проекта Symfony.

Прежде чем начать
-----------------

Мы начнём с добавления формы обратной связи к приложению Symfony.

Рисунок 1-1 показывает форму так, как её видит пользователь, который хочет отправить сообщение.

Рисунок 1-1 - Форма обратной связи

![Форма обратной связи](/images/forms_book/en/01_01.png "Форма обратной связи")

Для этой формы мы создадим три поля: имя пользователя, его электронная почта и сообщение, которое пользователь хочет отправить. В нашем примере мы просто отобразим отпраленную через форму информацию так, как показано на Рисунке 1-2.

Рисунок 1-2 - Страница с благодарностью

![Страница с благодарностью](/images/forms_book/en/01_02.png "Страница с благодарностью")

Рисунок 1-3 - Взаимодействие между приложением и пользователем

![Схема взаимодействия с пользователем](/images/forms_book/en/01_03.png "Схема взаимодействия с пользователем")

Виджеты
-------

### Классы `sfForm` и `sfWidget`

Пользователи вводят информацию в поля, тем самым заполняя формы. В Symfony, форма -- это объект, унаследованный от класса `sfForm`. В нашем примере, мы создадим класс `ContactForm`.

>**Note**
>`sfForm` -- это базовый класс для всех форм, который упрощает их конфигурирование и управление жизненным циклом ваших форм.

Вы можете начать конфигурироване вашей формы, добавив **виджеты** в методе `configure()`.

**Виджеты** представляют поля формы. Для нашего примера, нам необходимо добавить три виджета, которые будут представлять три наших поля: `name`, `email` и `message`. Листинг 1-1 показывает первую реализацию класса `ContactForm`.

Листинг 1-1 - Класс `ContactForm` с тремя полями

    [php]
    // lib/form/ContactForm.class.php
    class ContactForm extends BaseForm
    {
      public function configure()
      {
        $this->setWidgets(array(
          'name'    => new sfWidgetFormInputText(),
          'email'   => new sfWidgetFormInputText(),
          'message' => new sfWidgetFormTextarea(),
        ));
      }
    }

>**NOTE**
>В этой книге, мы никогда не указываем конструкцию `<?php`
>в примерах кода, который содержит чистый PHP, для экономии
>места и сохранении нескольких деревьев. Вы всегда должны
>помнить о необходимости добавления этой конструкции при
>создании нового PHP-файла.

Виджеты определяются в методе `configure()`. Этот метод автоматически вызывается из конструктора класса `sfForm`.

Метод `setWidgets()` используется для определения виджетов формы. Параметром метода `setWidgets()` является ассоциативный массив, в котором ключами являются имена полей, а значениями -- объекты виджетов. Каждый виджет является объектом, унаследованным от класса `sfWidget`. В нашем примере мы используем два типа виджетов:

  * `sfWidgetFormInputText`: Этот виджет представляет поле `input`
  * `sfWidgetFormTextarea`: Этот виджет представляет поле `textarea`

>**Note**
>По соглашению, мы сохраняем классы форм в каталоге `lib/form/`. Вы можете сохранять их в любом каталоге, который управляется механизмом автозагрузки Symfony, но как вы увидите позднее, Symfony использует каталог `lib/form/` для генерации форм из объектов модели.

### Отображение форм

Теперь наша форма готова к использованию. Сейчас мы создадим модуль Symfony для отображения формы:

    $ cd ~/PATH/TO/THE/PROJECT
    $ php symfony generate:module frontend contact

В модуле `contact`, изменим действие `index` таким образом, чтобы передать экземпляр формы в шаблон так, как показано в Листинге 1-2.

Листинг 1-2 - Класс действий из модуля `contact`

    [php]
    // apps/frontend/modules/contact/actions/actions.class.php
    class contactActions extends sfActions
    {
      public function executeIndex()
      {
        $this->form = new ContactForm();
      }
    }

При создании формы метод `configure()`, определённый ранее, будет вызван автоматически.

Теперь нам нужно создать шаблон для отображения формы так, как показано в Листинге 1-3.

Листинг 1-3 - Шаблон, отображающий форму

    [php]
    // apps/frontend/modules/contact/templates/indexSuccess.php
    <form action="<?php echo url_for('contact/submit') ?>" method="POST">
      <table>
        <?php echo $form ?>
        <tr>
          <td colspan="2">
            <input type="submit" />
          </td>
        </tr>
      </table>
    </form>

Форма Symfony обрабатывает только информацию о виджетах, отображающих информацию пользователем. В шаблоне `indexSuccess`, строка `<?php echo $form ?>` отображает только три поля. Другие элементы, такие как тэг `form` и кнопка отправки должны быть добавлены разработчиком. На первый взгляд это может выглядеть нелогичным, однако позднее вы увидите как благодаря этому можно легко встраивать одни формы в другие.

Использование конструкции `<?php echo $form ?>` весьма полезно при создании прототипов и определении форм. Она позволяет разработчикам сконцетрироваться на бизнес-логике и не беспокоиться о визуальных аспектах. В третьей главе будет рассказано, как настроить шаблон и макет формы.

>**Note**
При отображении объектов с помощью `<?php echo $form ?>`, движок PHP фактически будет выводить текстовое представление объекта `$form`. Для преобразованию объекта в строку PHP попытается вызвать магический метод `__toString()`. Каждый виджет реализует этот магический метод для преобразования объекта в HTML-код. Вызов `<?php echo $form ?>` эквивалентен вызову `<?php echo $form->__toString() ?>`.

Теперь мы можем увидеть форму в браузере (Рисунок 1-4) и проверить результат, введя в адресной строке действие `contact/index` (`/frontend_dev.php/contact`).

Рисунок 1-4 - Сгенерированная форма обратной связи

![Сгенерированная форма](/images/forms_book/en/01_04.png "Сгенерированная форма")

Листинг 1-4 демонстрирует сгенерированный шаблоном код.

    [html]
    <form action="/frontend_dev.php/contact/submit" method="POST">
      <table>
        
        <!-- Beginning of generated code by <?php echo $form ?> -->
        <tr>
          <th><label for="name">Name</label></th>
          <td><input type="text" name="name" id="name" /></td>
        </tr>
        <tr>
          <th><label for="email">Email</label></th>
          <td><input type="text" name="email" id="email" /></td>
        </tr>
        <tr>
          <th><label for="message">Message</label></th>
          <td><textarea rows="4" cols="30" name="message" id="message"></textarea></td>
        </tr>
        <!-- End of generated code by <?php echo $form ?> -->

        <tr>
          <td colspan="2">
            <input type="submit" />
          </td>
        </tr>
      </table>
    </form>

Мы видим, что форма отображается в трёх строках `<tr>` HTML-таблицы. Именно поэтому мы должны были заключить её вывод тэг <table>. Каждая строка включает тэг <label> и тэг формы ( <input> или <textarea> ).

### Метки

Метки всех полей генерируются автоматически. По умолчанию, метки являются трансформацией имени поля, при этом используются два правила: первая буква становится заглавной и все подчёркивания заменяются на пробелы. 
Если имя поля заканчивается на "_id", суффикс удаляется из метки.

Пример:

    [php]
    $this->setWidgets(array(
      'first_name' => new sfWidgetFormInputText(), // сгенерированная метка: "First name"
      'last_name'  => new sfWidgetFormInputText(), // сгенерированная метка: "Last name"
      'author_id'  => new sfWidgetFormInputText(), // сгенерированная метка: "Author"
    ));
 
Автоматически сгенерированные метки весьма полезны, однако фреймворк позволяет вам определить свои собственные метки через метод `setLabels()`:

    [php]
    $this->widgetSchema->setLabels(array(
      'name'    => 'Ваше имя',
      'email'   => 'Ваша электропочта',
      'message' => 'Ваше сообщение',
    ));

Также вы можете изменить конкретную метку через вызов `setLabel()`:

    [php]
    $this->widgetSchema->setLabel('email', 'Ваша электропочта');

Наконец, как вы увидите в Главе 3, вы можете расширять метки из шаблона для дальнейшей настройки формы.

>**Sidebar**
>Схема виджетов
>
>Когда мы используем метод `setWidgets()`, Symfony создаёт объект `sfWidgetFormSchema`. Этот объект является виджетом, который позволяет вам представить набор виджетов. В нашей форме `ContactForm`, мы вызываем метод `setWidgets()`. Это эквивалентно следующему коду:
>
>     [php]
>     $this->setWidgetSchema(new sfWidgetFormSchema(array(
>       'name'    => new sfWidgetFormInputText(),
>       'email'   => new sfWidgetFormInputText(),
>       'message' => new sfWidgetFormTextarea(),
>     )));
>
>     // полностью эквивалентно:
>
>     $this->widgetSchema = new sfWidgetFormSchema(array(
>       'name'    => new sfWidgetFormInputText(),
>       'email'   => new sfWidgetFormInputText(),
>       'message' => new sfWidgetFormTextarea(),
>     ));
>
>Метод `setLabels()` применяется к коллекции виджетов, заключённой в объект `widgetSchema`.
>
>Мы увидим в Главе 5, что понятие "схема виджетов" упрощает управление встроенными формами.

### Вместо генерации таблиц

Даже если форма по умолчанию отображается как HTML-таблица, формат макета может быть изменён. Различные типы макетов определяются в классах, которые наследуются от `sfWidgetFormSchemaFormatter`. По умолчанию, форма использует формат `table`, определённый в классе `sfWidgetFormSchemaFormatterTable`. Также вы можете использовать формат `list`:

    [php]
    class ContactForm extends BaseForm
    {
      public function configure()
      {
        $this->setWidgets(array(
          'name'    => new sfWidgetFormInputText(),
          'email'   => new sfWidgetFormInputText(),
          'message' => new sfWidgetFormTextarea(),
        ));

        $this->widgetSchema->setFormFormatterName('list');
      }
    }

Эти два формата существуют по умолчанию, а в Главе 5 вы увидите, как создать свой собственный класс формата. Теперь, когда мы знаем как отобразить форму, давайте посмотрим как управлять отпраляемыми данными.

### Отправка формы

Когда мы создавали шаблон для отображения формы, мы использовали внутренний URL `contact/submit` в тэге `form` для отправки формы. Теперь нам необходимо добавить действие `submit` в модуль `contact`. Листинг 1-5 показывает как действие может получить информацию от пользователя и отправить её на страницу `Спасибо`, где мы просто отобразим её обратно пользователю.

Листинг 1-5 - Использование действия `submit` в модуле `contact`

    [php]
    public function executeSubmit($request)
    {
      $this->forward404Unless($request->isMethod('post'));

      $params = array(
        'name'    => $request->getParameter('name'),
        'email'   => $request->getParameter('email'),
        'message' => $request->getParameter('message'),
      );

      $this->redirect('contact/thankyou?'.http_build_query($params));
    }

    public function executeThankyou()
    {
    }

    // apps/frontend/modules/contact/templates/thankyouSuccess.php
    <ul>
      <li>Name:    <?php echo $sf_params->get('name') ?></li>
      <li>Email:   <?php echo $sf_params->get('email') ?></li>
      <li>Message: <?php echo $sf_params->get('message') ?></li>
    </ul>

>**Note**
>`http_build_query` -- это встроенная PHP-функция, которая генерирует URL-закодированную строку запроса из массива параметров.

Метод `executeSubmit()` выполняет три действия:

  * По причинам безопасности, мы проверяем что страница отправляется через HTTP-метод `POST`. Если отправлено не через метод `POST`, то пользователь перенаправляется на страницу 404. В шаблоне `indexSuccess`, мы определяем метод отправки как `POST` (`<form ... method="POST">`):

        [php]
        $this->forward404Unless($request->isMethod('post'));

  * Далее мы получаем введённые пользователем значения и сохраняем их массиве `params`:

        [php]
        $params = array(
          'name'    => $request->getParameter('name'),
          'email'   => $request->getParameter('email'),
          'message' => $request->getParameter('message'),
        );

  * Наконец, мы перенаправляем пользователя на страницу `Спасибо`  (`contact/thankyou`), которая отобразит информацию:

        [php]
        $this->redirect('contact/thankyou?'.http_build_query($params));

Вместо перенаправления пользователя на другую страницу, мы могли бы создать шаблон `submitSuccess.php`. Несмотря на это, рекомендуется всегда перенаправлять пользователя после выполнения запроса с методом `POST`:

  * Это позволяет исключить повторную отправку формы, если пользователь перезагрузит страницу `Спасибо`.

  * Пользователь также может нажать кнопку "Назад" без получения всплывающего сообщения о повторной отправке формы.

>**Tip**
>Вы могли заметить, что прототип метода `executeSubmit()` отличается от `executeIndex()`. При вызове этих методов, Symfony передаёт текущий объект `sfRequest` в качестве первого аргумента методов `executeXXX()`. Однако в PHP у вас нет необходимости указывать все параметры, именно из-за этого мы не определили переменную `request` в методе `executeIndex()`.

Рисунок 1-5 показывает вызываемые методы при взаимодействии с пользователем.

Рисунок 1-5 - Вызываемые методы

![Вызываемые методы](/images/forms_book/en/01_05.png "Вызываемые методы")

>**Note**
>При отображении пользовательского ввода в шаблоне, у нас есть риск нарваться на атаку XSS (Cross-Site Scripting). Более подробную информацию о предотвращении XSS через реализацию стратегии экранирования можно получить в главе [Inside the View Layer](http://www.symfony-project.org/book/1_2/07-Inside-the-View-Layer#chapter_07_output_escaping) книги "The Definitive Guide to symfony".

После отправки вами формы, вы должны увидеть страницу аналогичную Рисунку 1-6.

Рисунок 1-6 - Страница, отображаемая после отправки формы

![Страница, отображаемая после отправки формы](/images/forms_book/en/01_06.png "Страница, отображаемая после отправки формы")

Instead of creating the `params` array, it would be easier to get the information from the user directly in an array. Listing 1-6 modifies the `name` HTML attribute from widgets to store the field values in the `contact` array.

Listing 1-6 - Modification of the `name` HTML attribute from widgets

    [php]
    class ContactForm extends BaseForm
    {
      public function configure()
      {
        $this->setWidgets(array(
          'name'    => new sfWidgetFormInputText(),
          'email'   => new sfWidgetFormInputText(),
          'message' => new sfWidgetFormTextarea(),
        ));

        $this->widgetSchema->setNameFormat('contact[%s]');
      }
    }

Calling `setNameFormat()` allows us to modify the `name` HTML attribute for all widgets. `%s` will automatically be replaced by the name of the field when generating the form. For example, the `name` attribute will then be `contact[email]` for the `email` field. PHP automatically creates an array with the values of a request including a `contact[email]` format. This way the field values will be available in the `contact` array.

We can now directly get the `contact` array from the `request` object as shown in Listing 1-7.

Listing 1-7 - New format of the `name` attributes in the action widgets

    [php]
    public function executeSubmit($request)
    {
      $this->forward404Unless($request->isMethod('post'));

      $this->redirect('contact/thankyou?'.http_build_query($request->getParameter('contact')));
    }

When displaying the HTML source of the form, you can see that symfony has generated a `name` attribute depending not only on the field name and format, but also an `id` attribute. The `id` attribute is automatically created from the `name` attribute by replacing the forbidden characters by underscores (`_`):

  | **Name**  | **Attribute `name`** | **Attribute `id`**  |
  | --------- | -------------------- | ------------------- |
  | name      | contact[name]        | contact_name        |
  | email     | contact[email]       | contact_email       |
  | message   | contact[message]     | contact_message     |

### Another solution

In this example, we used two actions to manage the form: `index` for the display, `submit` for the submit. Since the form is displayed with the `GET` method and submitted with the `POST` method, we can also merge the two methods in the `index` method as shown in Listing 1-8.

Listing 1-8 - Merging of the two actions used in the form

    [php]
    class contactActions extends sfActions
    {
      public function executeIndex($request)
      {
        $this->form = new ContactForm();

        if ($request->isMethod('post'))
        {
          $this->redirect('contact/thankyou?'.http_build_query($request->getParameter('contact')));
        }
      }
    }

You also need to change the form `action` attribute in the `indexSuccess.php` template:

    [php]
    <form action="<?php echo url_for('contact/index') ?>" method="POST">

As we will see later, we prefer to use this syntax since it is shorter and makes the code more coherent and understandable.

Configuring the Widgets
-----------------------

### Widgets options

If a website is managed by several webmasters, we would certainly like to add a drop-down list with themes in order to redirect the message according to what is asked (Figure 1-7). Listing 1-9 adds a `subject` with a drop-down list using the `sfWidgetFormSelect` widget.

Figure 1-7 - Adding a `subject` Field to the Form

![Adding a `subject` Field to the Form](/images/forms_book/en/01_07.png "Adding a `subject` Field to the Form")

Listing 1-9 - Adding a `subject` Field to the Form

    [php]
    class ContactForm extends BaseForm
    {
      protected static $subjects = array('Subject A', 'Subject B', 'Subject C');

      public function configure()
      {
        $this->setWidgets(array(
          'name'    => new sfWidgetFormInputText(),
          'email'   => new sfWidgetFormInputText(),
          'subject' => new sfWidgetFormSelect(array('choices' => self::$subjects)),
          'message' => new sfWidgetFormTextarea(),
        ));

        $this->widgetSchema->setNameFormat('contact[%s]');
      }
    }

>**SIDEBAR**
>The `choices` option of the `sfWidgetFormSelect` Widget
>
>PHP does not make any distinction between an array and an associative array, so the array we used for the subject list is identical to the following code:
>
>     [php]
>     $subjects = array(0 => 'Subject A', 1 => 'Subject B', 2 => 'Subject C');
>
>The generated widget takes the array key as the `value` attribute of the `option` tag, and the related value as content of the tag:
>
>     [php]
>     <select name="contact[subject]" id="contact_subject">
>       <option value="0">Subject A</option>
>       <option value="1">Subject B</option>
>       <option value="2">Subject C</option>
>     </select>
>
>In order to change the `value` attributes, we just have to define the array keys:
>
>     [php]
>     $subjects = array('A' => 'Subject A', 'B' => 'Subject B', 'C' => 'Subject C');
>
>Which generates the HTML template:
>
>     [php]
>     <select name="contact[subject]" id="contact_subject">
>       <option value="A">Subject A</option>
>       <option value="B">Subject B</option>
>       <option value="C">Subject C</option>
>     </select>

The `sfWidgetFormSelect` widget, like all widgets, takes a list of options as the first argument. An option may be mandatory or optional. The `sfWidgetFormSelect` widget has a mandatory option, `choices`. Here are the available options for the widgets we already used:

  | **Widget**             | **Mandatory Options** | **Additional Options**           |
  | ---------------------- | --------------------- | -------------------------------- |
  | `sfWidgetFormInput`    | -                     | `type` (default to `text`)       |
  |                        |                       | `is_hidden` (default to `false`) |
  | `sfWidgetFormSelect`   | `choices`             | `multiple` (default to `false`)  |
  | `sfWidgetFormTextarea` | -                     | -                                |

>**Tip**
>If you want to know all of the options for a widget, you can refer to the complete API documentation available online at  ([http://www.symfony-project.org/api/1_2/](http://www.symfony-project.org/api/1_2/)). All of the options are explained, as well as the additional options default values. For instance, all of the options for the `sfWidgetFormSelect` are available here: ([http://www.symfony-project.org/api/1_2/sfWidgetFormSelect](http://www.symfony-project.org/api/1_2/sfWidgetFormSelect)).

### The Widgets HTML Attributes

Each widget also takes a list of HTML attributes as second optional argument. This is very helpful to define default HTML attributes for the generated form tag. Listing 1-10 shows how to add a `class` attribute to the `email` field.

Listing 1-10 - Defining Attributes for a Widget

    [php]
    $emailWidget = new sfWidgetFormInputText(array(), array('class' => 'email'));

    // Generated HTML
    <input type="text" name="contact[email]" class="email" id="contact_email" />

HTML attributes also allow us to override the automatically generated identifier, as shown in Listing 1-11.

Listing 1-11 - Overriding the `id` Attribute

    [php]
    $emailWidget = new sfWidgetFormInputText(array(), array('class' => 'email', 'id' => 'email'));

    // Generated HTML
    <input type="text" name="contact[email]" class="email" id="email" />

It is even possible to set default values to the fields using the `value` attribute as Listing 1-12 shows.

Listing 1-12 - Widgets Default Values via HTML Attributes

    [php]
    $emailWidget = new sfWidgetFormInputText(array(), array('value' => 'Your Email Here'));

    // Generated HTML
    <input type="text" name="contact[email]" value="Your Email Here" id="contact_email" />

This option works for `input` widgets, but is hard to carry through with `checkbox` or `radio` widgets, and even impossible with a `textarea` widget. The `sfForm` class offers specific methods to define default values for each field in a uniform way for any type of widget.


>**Note**
>We recommend to define HTML attributes inside the template and not in the form itself (even if it is possible) to preserve the layers of separation as we will see in Chapter three.

### Defining Default Values For Fields

It is often useful to define a default value for each field. For instance, when we display a help message in the field that disappears when the user focuses on the field. Listing 1-13 shows how to define default values via the `setDefault()` and `setDefaults()` methods.

Listing 1-13 - Default Values of the Widgets via the `setDefault()` and `setDefaults()` Methods

    [php]
    class ContactForm extends BaseForm
    {
      public function configure()
      {
        // ...

        $this->setDefault('email', 'Your Email Here');

        $this->setDefaults(array('email' => 'Your Email Here', 'name' => 'Your Name Here'));
      }
    }

The `setDefault()` and `setDefaults()` methods are very helpful to define 
identical default values for every instance of the same form class. If we
want to modify an existing object using a form, the default values will 
depend on the instance, therefore they must be dynamic. Listing 1-14 shows
the `sfForm` constructor has a first argument that sets default values 
dynamically.

Listing 1-14 - Default Values of the Widgets via the Constructor of `sfForm`

    [php]
    public function executeIndex($request)
    {
      $this->form = new ContactForm(array('email' => 'Your Email Here', 'name' => 'Your Name Here'));

      // ...
    }

>**SIDEBAR**
>Protection XSS (Cross-Site Scripting)
>
>When setting HTML attributes for widgets, or defining default values, the `sfForm` class automatically protects these values against XSS attacks during the generation of the HTML code. This protection does not depend on the `escaping_strategy` configuration of the `settings.yml` file. If a content has already been protected by another method, the protection will not be applied again.
>
>It also protects the `'` and `"` characters that might invalidate the generated HTML.
>
>Here is an example of this protection:
>
>     [php]
>     $emailWidget = new sfWidgetFormInputText(array(), array(
>       'value' => 'Hello "World!"',
>       'class' => '<script>alert("foo")</script>',
>     ));
>     
>     // Generated HTML
>     <input
>       value="Hello &quot;World!&quot;"
>       class="&lt;script&gt;alert(&quot;foo&quot;)&lt;/script&gt;"
>       type="text" name="contact[email]" id="contact_email"
>     />
