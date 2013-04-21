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

Вместо создания массива `params`, гораздо проще будет получить информацию от пользователя сразу в массиве. Листинг 1-6 изменяет HTML-атрибут `name` виджетов таким образом, что вводимые значения сохраняются в массиве `contact`.

Листинг 1-6 - Модификация HTML-атрибута `name` из виджетов

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

Вызов `setNameFormat()` позволяет нам изменить HTML-атрибут `name` для всех виджетов. `%s` автоматически заменяется на имя поля при генерации формы. Например, для поля `email` атрибут `name` будет `contact[email]`. PHP автоматически создаёт массив из включённых в запрос значений. Таким образом, значения полей будут доступны в массиве `contact`.

Теперь мы можем напрямую получить массив `contact` из объекта `request` как показано в Листинге 1-7.

Листинг 1-7 - Новый формат атрибутов `name` виджетов в действии

    [php]
    public function executeSubmit($request)
    {
      $this->forward404Unless($request->isMethod('post'));

      $this->redirect('contact/thankyou?'.http_build_query($request->getParameter('contact')));
    }

При просмотре исходного HTML-кода формы вы можете видеть, что Symfony генерирует не только атрибут `name` в зависимости от имени и формата, но и атрибут `id`. Атрибут `id` автоматически создаётся из атрибута `name` заменой недопустимых символов на подчёркивания (`_`):

  | **Имя**  | **Атрибут `name`** | **Атрибут `id`**  |
  | --------- | ----------------- | ----------------- |
  | name      | contact[name]     | contact_name      |
  | email     | contact[email]    | contact_email     |
  | message   | contact[message]  | contact_message   |

### Другое решение

В этом примере мы используем два действия для управления формой: `index` для отображения и `submit` для обработки присланных данных. Так как форма отображается с методом `GET` и отправляется с методом `POST`, мы можем объединить эти два метода в действии `index`, как показано в Листинге 1-8.

Листинг 1-8 - Объединение двух используемых формой действий

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

Также вам необходимо изменить атрибут `action` формы в шаблоне `indexSuccess.php`:

    [php]
    <form action="<?php echo url_for('contact/index') ?>" method="POST">

Как мы увидим позже, мы предпочитаем использовать такой синтаксис, поскольку он короче и делает код более последовательным и понятным.

Конфигурирование виджетов
-------------------------

### Опции виджетов

Если сайт находится в ведении нескольких веб-мастеров, нам конечно же захочется добавить выпадающий список с темами, чтобы перенаправлять сообщения соответственно вопросу (Рисунок 1-7). Листинг 1-9 добавляет поле `subject` с выпадающим списком, используя виджет `sfWidgetFormSelect`.

Рисунок 1-7 - Добавление к форме поля `subject`

![Добавление поля `subject`](/images/forms_book/en/01_07.png "Добавление поля `subject`")

Листинг 1-9 - Добавление к форме поля `subject`

    [php]
    class ContactForm extends BaseForm
    {
      protected static $subjects = array('Тема А', 'Тема Б', 'Тема В');

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
>Опция `choices` виджета `sfWidgetFormSelect`
>
>PHP не разделяет массивы и ассоциативные массивы, поэтому использованный нами массив со списком тем идентичен следующему коду:
>
>     [php]
>     $subjects = array(0 => 'Тема А', 1 => 'Тема Б', 2 => 'Тема В');
>
>При выводе виджета ключ в массиве становится атрибутом `value` тэга `option`, а соответствующее значение помещается в содержимое тэга:
>
>     [php]
>     <select name="contact[subject]" id="contact_subject">
>       <option value="0">Тема А</option>
>       <option value="1">Тема Б</option>
>       <option value="2">Тема В</option>
>     </select>
>
>Для изменения атрибутов `value`, нам необходимо просто определить ключи массива:
>
>     [php]
>     $subjects = array('А' => 'Тема А', 'Б' => 'Тема Б', 'В' => 'Тема В');
>
>Это приведёт к генерации следующего HTML:
>
>     [php]
>     <select name="contact[subject]" id="contact_subject">
>       <option value="А">Тема А</option>
>       <option value="Б">Тема Б</option>
>       <option value="В">Тема В</option>
>     </select>

Виджет `sfWidgetFormSelect`, как и все виджеты, принимает список опцив в качестве первого аргумента. Опции могут быть обязательными и нет. В виджете `sfWidgetFormSelect` обязательной опцией является `choices`. Вот доступные опции для использованных нами виджетов:

  | **Виджет**             | **Обязательные опции** | **Additional Options**             |
  | ---------------------- | ---------------------- | ---------------------------------- |
  | `sfWidgetFormInput`    | -                      | `type` (по уполчанию `text`)       |
  |                        |                        | `is_hidden` (по уполчанию `false`) |
  | `sfWidgetFormSelect`   | `choices`              | `multiple` (по уполчанию `false`)  |
  | `sfWidgetFormTextarea` | -                      | -                                  |

>**Tip**
>Если вам интересно знать обо всех опциях виджетов, вы можете найти их в документации по API, доступной онлайн по адресу  ([http://www.symfony-project.org/api/1_4/](http://www.symfony-project.org/api/1_4/)).
>Описаны все опции, а также приведены значения по умолчанию для необязательных опций.
>Например, все опции `sfWidgetFormSelect` доступны здесь: ([http://www.symfony-project.org/api/1_4/sfWidgetFormSelect](http://www.symfony-project.org/api/1_4/sfWidgetFormSelect)).

### HTML-атрибуты виджетов

Каждый виджет также принимает список HTML-атрибутов вторым необязательным аргументом. Это очень полезно для определения заданных по умолчанию HTML-атрибутов, которые будут использованы при генерации тэга формы. Листинг 1-10 показывает как добавить атрибут `class` к полю `email`.

Листинг 1-10 - Определение атрибутов для виджета

    [php]
    $emailWidget = new sfWidgetFormInputText(array(), array('class' => 'email'));

    // Генерируемый HTML
    <input type="text" name="contact[email]" class="email" id="contact_email" />

HTML-атрибуты также позволяют нам переопределить генерируемый по умолчанию идентификатор, см. Листинг 1-11.

Листинг 1-11 - Переопределение атрибута `id`

    [php]
    $emailWidget = new sfWidgetFormInputText(array(), array('class' => 'email', 'id' => 'email'));

    // Генерируемый HTML
    <input type="text" name="contact[email]" class="email" id="email" />

Также возможно установить для поля значение по умолчанию через атрибут `value`, см. Листинг 1-12.

Листинг 1-12 - Значения виджета по умолчанию через HTML-атрибуты

    [php]
    $emailWidget = new sfWidgetFormInputText(array(), array('value' => 'Your Email Here'));

    // Генерируемый HTML
    <input type="text" name="contact[email]" value="Your Email Here" id="contact_email" />

Этот способ работает для виджетов `input`, но его трудно реализовать для виджетов `checkbox` или `radio`, и совсем невозможно для виджетов `textarea`. Класс `sfForm` предоставляет специальные методы для определения значений по умолчанию для каждого поля унифицированным способом для любого типа виджетов.


>**Note**
>Мы рекомендуем определять HTML-атрибуты прямо в шаблоне, а не в самой форме (хотя это и возможно), для сохранения разделения на слои как будет показано в Главе 3.

### Задание полям значений по умолчанию

Часто бывает полезно определить значение по умолчанию для каждого поля. Например, когда мы выводим справочное сообщение в поле, которое исчезает, если пользователь перемещает фокус в поле. Листинг 1-13 показывает, как определить значение по умолчанию через методы `setDefault()` и `setDefaults()`.

Листинг 1-13 - Значения по умолчанию для виджетов через методы `setDefault()` и `setDefaults()`

    [php]
    class ContactForm extends BaseForm
    {
      public function configure()
      {
        // ...

        $this->setDefault('email', 'Your Email Here');

        $this->setDefaults(array('email' => 'Введите здесь ваш E-Mail', 'name' => 'Введите здесь ваше имя'));
      }
    }

Методы `setDefault()` и `setDefaults()` крайне полезны для определения идентичных значений по умолчанию для всех экземпляров одного и того же класса формы.
Если же мы хотим изменять существующие объекты с помощью форм, значения по умолчанию должны зависеть от экземпляра, т.е. быть динамическими.
В Листинге 1-14 демонстрируется приём в первом аргументе конструктора `sfForm` таких динамичных значений по умолчанию.

Листинг 1-14 - Значения по умолчанию для виджетов через конструктор `sfForm`

    [php]
    public function executeIndex($request)
    {
      $this->form = new ContactForm(array('email' => 'Your Email Here', 'name' => 'Your Name Here'));

      // ...
    }

>**SIDEBAR**
>Защита от XSS (Cross-Site Scripting)
>
>При установке HTML-атрибутов или определении значений по умолчанию для виджетов, класс `sfForm` автоматически защищает эти значения от XSS-атак при генерации HTML-кода. Эта защита не зависит от конфигурации `escaping_strategy` в файле `settings.yml`. Если содержимое уже защищено другим методом, то повторно защита не применяется.
>
>Также экранируются символы `'` и `"`, которые могут привести генерируемый HTML в невалидное состояние.
>
>Вот пример работы защиты:
>
>     [php]
>     $emailWidget = new sfWidgetFormInputText(array(), array(
>       'value' => 'Hello "World!"',
>       'class' => '<script>alert("foo")</script>',
>     ));
>     
>     // Сгенерированный HTML
>     <input
>       value="Hello &quot;World!&quot;"
>       class="&lt;script&gt;alert(&quot;foo&quot;)&lt;/script&gt;"
>       type="text" name="contact[email]" id="contact_email"
>     />
