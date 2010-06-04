Capitolo 4 - Le basi per la creazione delle pagine
==================================================

Curiosamente, il primo tutorial che i programmatori seguono quando devono imparare una nuovo linguaggio o un framework � quello che permette di visualizzare "Buongiorno, mondo!" sullo schermo. E' strano pensare al computer come un qualcosa che pu� salutare il mondo intero, dal momento che ogni tentativo nel campo dell'intelligenza artificiale ha finora ottenuto scarse capacit� di conversazione. Ma symfony non � pi� stupido di qualsiasi altro programma, e la prova � che � possibile creare una pagina che dice "Buongiorno, `<Il Tuo Nome>`".

Questo capitolo spiegher� come creare un modulo, che � un elemento strutturale che raggruppa le pagine. Si potr� anche imparare a creare una pagina, che � divisa in una azione e un template, a causa dell'utilizzo del pattern MVC. Collegamenti e form sono le interazioni web di base; si vedr� come inserirli in un template e gestirli in un'azione.

Creare lo scheletro di un modulo
--------------------------------

Come spiegato nel capitolo 2, symfony raggroppa le pagine in moduli. Prima di creare una pagina, bisogna creare un modulo, che inizialmente � una struttura vuota di file che symfony pu� riconoscere.

La riga di comando di symfony automatizza la creazione di moduli. Basta eseguire il task `generate:module` con il nome dell'applicazione e il nome del modulo cmoe argomenti. Nel precedente capitolo, � stata creata l'applicazione `frontend`. Per aggiungere un modulo `contenuto` a questa applicazione, digitare i seguenti comandi:

    $ cd ~/mioprogetto
    $ php symfony generate:module frontend contenuto

    >> dir+      ~/mioprogetto/apps/frontend/modules/contenuto/actions
    >> file+     ~/mioprogetto/apps/frontend/modules/contenuto/actions/actions.class.php
    >> dir+      ~/mioprogetto/apps/frontend/modules/contenuto/templates
    >> file+     ~/mioprogetto/apps/frontend/modules/contenuto/templates/indexSuccess.php
    >> file+     ~/mioprogetto/test/functional/frontend/contenutoActionsTest.php
    >> tokens    ~/mioprogetto/test/functional/frontend/contenutoActionsTest.php
    >> tokens    ~/mioprogetto/apps/frontend/modules/contenuto/actions/actions.class.php
    >> tokens    ~/mioprogetto/apps/frontend/modules/contenuto/templates/indexSuccess.php

A parte le cartelle `actions/` e `templates/`, questo comando crea solo tre file. Il primo si trova nella cartella `test/`, riguarda i test funzionali e non si ha bisogno di farci caso fino al Capitolo 15. `actions.class.php` (mostrato nel Listato 4-1) esegue il forward al modulo predefinito con la pagina di congratulazioni. Il file `templates/indexSuccess.php` � vuoto.

Listato 4-1 - L'azione predefinita generata in `actions/actions.class.php`

    [php]
    <?php

    class contenutoActions extends sfActions
    {
      public function executeIndex()
      {
        $this->forward('default', 'module');
      }
    }

>**NOTE**
>Se si guarda nel file `actions.class.php`, si troveranno altre righe, compresi molti commenti. Questo perch� symfony consiglia l'uso dei commenti PHP per documentare i progetti e predispone ciascun file con classi ad essere compatibile con lo [strumento phpDocumentor](http://www.phpdoc.org/).

Per ogni nuovo modulo, symfony crea una azione `index` predefinita. Questa si compone di un metodo azione chiamato `executeIndex` e un file template chiamato `indexSuccess.php`. Il significato del prefisso `execute` e del suffisso `Success` verr� spiegato rispettivamente nei Capitoli 6 e 7. Nel frattempo, si pu� considerare che questa nomenclatura � una convenzione. Si pu� vedere la pagina corrispondente (riprodotta in Figura 4-1) aprendo il seguente URL:

    http://localhost/frontend_dev.php/contenuto/index

L'azione predefinita `index` non verr� usata in questo capitolo, quindi si pu� rimuovere il metodo `executeIndex()` dal file `actions.class.php` e cancellare il file `indexSuccess.php` dalla cartella `templates/`.

>**NOTE**
>Symfony mette a disposizione altri modi per creare un modulo oltre che da riga di comando. Uno di questi � quello di creare le cartelle e i file manualmente. In molti casi, le azioni e i template di un modulo hanno lo scopo di manipolare i dati di una determinata tabella. Essendo che il codice necessario per creare, recuperare, aggiornare ed eliminare record da una tabella � spesso lo stesso, symfony fornisce un meccanismo per generare tale codice.

Figura 4-1 - La pagina index creata in modo predefinito

![La pagina index creata in modo predefinito](http://www.symfony-project.org/images/book/1_4/F0401.jpg "The default generated index page")

Aggiungere una pagina
---------------------

In symfony, la logica che sta dietro alle pagine � memorizzata nell'azione, mentre la presentazione � nei template. Le pagine senza logica ichiedono (ancora) una azione vuota.

### Aggiungere un'azione

La pagina "Buongiorno, mondo!" sar� accessibile attravero un'azione `show`. Per crearla, � sufficiente aggiungere un metodo `executeShow` alla classe `contenutoActions`, come mostrato nel Listato 4-2.

Listato 4-2 - Aggiungere un'azione vuol dire aggiungere un metodo Execute alla classe Action

    [php]
    <?php

    class contenutoActions extends sfActions
    {
      public function executeShow()
      {
      }
    }

Il nome del metodo dell'azione � sempre `executeXxx()`, dove la seconda parte del nome � il nome dell'azione con la prima lettera in maiuscolo.

Ora, se si accede al seguente URL:

    http://localhost/frontend_dev.php/contenuto/show

symfony si lamenter� per il fatto che manca il template `showSuccess.php`. E' normale; in symfony, una pagina � sempre costituita da un'azione e da un template.

>**CAUTION**
>Le URL (non i nomi dominio) sono case-sensitive e quindi lo sono anche in symfony (anche se i nomi del metodi in PHP sono case-insensitive). Questo significa che symfony restituir� un errore 404 se si prova a visualizzare `sHow` con il browser.

-

>**SIDEBAR**
>Le URL sono parte della risposta
>
>Symfony ha un sistema di routing che permette di avere una separazione completa tra il nome dell'azione attuale e la struttura della URL necessaria per chiamarlo. Ci� consente la formattazione personalizzata della URL, come se fosse parte della risposta. Non si � pi� limitati dalla struttura del file, n� dai parametri della richiesta; l'URL per una azione pu� essere una frase che si desidera. Per esempio, la chiamata all'azione index di un modulo chiamato articolo, generalmente assomiglia a questa:
>
>     http://localhost/frontend_dev.php/article/index?id=123
>
>Questo URL recupera un determinato articolo da un database. In questo esempio, recupera un articolo (con `id=123`) nella sezione Europa che parla specificamente delle finanze in Francia. Ma l'URL pu� essere scritta in un modo completamente diverso con una semplice modifica nel file di configurazione `routing.yml`:
>
>     http://localhost/articles/europe/france/finance.html
>
>L'URL risultante diventa amica non solo dei motori di ricerca, ma anche dell'utente, che pu� utilizzare la barra degli indirizzi come linea di comando per fare pseudo query personalizzate, come si pu� vedere di seguito:
>
>     http://localhost/articles/tagged/finance+france+euro
>
>Symfony sa come fare il parse e generare URL intelligenti per l'utente. Il sistema per le rotte estrapola automaticamente i parametri della richiesta da una URL intelligente e li rende disponibili all'azione. Formatta anche gli hyperlink inclusi nella risposta in modo che vengano mostrati in modo "intelligente". Per saperne di pi� su questa caratteristica, leggere il Capitolo 9.
>
>Nel complesso, questo significa che il modo in cui si d� un nome alle azioni delle applicazioni non dovrebbe essere influenzato dal modo in cui scegliamo l'URL per chiamarli, ma dalle funzioni delle azioni nell'applicazione. Il nome di una azione deve descrivere cosa fa l'azione reale e spesso � spesso un verbo in forma all'infinito (come `show`, `list`, `edit` e cos� via). I nomi delle azioni possono essere del tutto invisibili all'utente finale, quindi non bisogna aver paura di utilizzare nomi di azioni espliciti (come `listByName` or `showWithComments`). In questo modo si pu� risparmiare sui commenti del codice atti a spiegare la funzione dell'azione, ed inoltre il codice sar� molto pi� facile da leggere.

### Aggiungere un template

L'azione si attende un template per visualizzare se stessa. Un template � un file localizzato nella cartella `templates/` di un modulo, nominato in base all'azione e alla azione di terminazione. L'azione di terminazione predefinita � "success", quindi il file del template che deve essere creato per l'azione `show` deve essere chiamato `showSuccess.php`.

I template devono contenere solo codice di presentazione, quindi al loro interno bisogna utilizzare meno codice PHP possibile. Una pagina che mostra la scritta "Buongiorno, mondo!" pu� avere un template tanto semplice quanto quello mostrato nel Listato 4-3.

Listato 4-3 - Il template `contenuto/templates/showSuccess.php`

    [php]
    <p>Buongiorno, mondo!</p>

Se � necessario eseguire del codice PHP nel template, bisogna evitare di utilizzare la normale sintassi PHP, come mostrato nel Listato 4-4. Per i template � meglio utilizzare la sintassi PHP alternativa, come mostrato nel Listato 4-5, in modo da rendere il codice pi� comprensibile anche per chi non � un programmatore PHP. Non solo il codice finale sar� perfettamente indentato, ma aiuter� anche a tenere il codice PHP complesso nell'azione, in quanto solo le istruzioni di controllo (`if`, `foreach`, `while` e cos� via) hanno una sintassi alternativa.

Listato 4-4 - L'suale sintassi PHP, buona per le azioni, ma cattiva per i template

    [php]
    <p>Buongiorno, mondo!</p>
    <?php

    if ($test)
    {
      echo "<p>".time()."</p>";
    }

    ?>

Listato 4-5 - La sintassi alternativa PHP, buona per i template

    [php]
    <p>Buongiorno, mondo!</p>
    <?php if ($test): ?>
      <p><?php echo time(); ?></p>
    <?php endif; ?>

>**TIP**
>Una buona regola per controllare se la sintassi del template � sufficientemente leggibile � quella che il file non deve contenere codice HTML stampato dall'echo di PHP o parentesi grafe. La maggior parte delle volte, quando si apre un `<?php`, verr� chiuso con `?>` nella stessa riga.

### Passare informazioni dall'azione al template

Il compito dell'azione � quello di fare tutte le elaborazioni complicate, il recupero dei dati, i controlli e assegnare variabili per il template che verranno visualizzate o verificate con istruzioni di controllo. Symfony rende gli attributi delle classi di azioni (accessibili attraverso `$this->nomeVariabile` nell'azione) direttamente accessibili al template nello spazio dei nomi globali (attravero `$nomeVariabile`). I Listati 4-6 e 4-7 mostrano come passare informazioni dall'azione al template.

Listato 4-6 - Impostare l'attributo di una azione nell'azione per renderlo disponibile nel template

    [php]
    <?php

    class contenutoActions extends sfActions
    {
      public function executeShow()
      {
        $today = getdate();
        $this->hour = $today['hours'];
      }
    }

Listato 4-7 - Il template ha accesso diretto agli attributi dell'azione

    [php]
    <p>Buongiorno, mondo!</p>
    <?php if ($hour >= 18): ?>
      <p>O dovrei dire buona sera? Sono gi� le <?php echo $hour ?>.</p>
    <?php endif; ?>

Notare che l'utilizzo del tag breve di apertura (`<?=`, equivalente a `<?php echo`) non � raccomandato per applicazioni web professionali, perch� il server web in produzione potrebbe essere in grado di interpretare pi� di un linguaggio di script e conseguentemente confondersi. Inoltre il tag di apertura breve non funziona con la configurazione PHP predefinita e necessita di modifiche sul server per essere attivata. In ultimo, quando si ha a che fare con l'XML e la validazione, si vengono a creare dei problemi, perch� `<?` in XML ha un significato speciale.

>**NOTE**
>Il template ha gi� accesso ad alcuni frammenti di dati senza la necessit� di assegnare ogni singola variabile nell'azione. Ogni template pu� chiamare metodi degli oggetti `$sf_request`, `$sf_params`, `$sf_response` e `$sf_user`. Contengono dati relativi alla richiesta corrente, ai parametri della richiesta, risposta e sessione. Presto verr� spiegato come utilizzarli in modo efficiente.

Link a un'altra azione
----------------------

Sappiamo gi� che c'� un disaccoppiamento totale tra il nome di una azione e l'URL utilizzata per chiamarla. Quindi se in un template si crea un link a `update` come nel Listato 4-8, funzioner� solo con le rotte in modalit� predefinita. Se successivamente si decide di cambiare il modo in cui le URL devono apparire, sar� necessario rivedere tutti i template per modificare i collegamenti ipertestuali.

Listato 4-8 - Collegamenti ipertestuali, il modo classico

    [php]
    <a href="/frontend_dev.php/contenuto/update?name=anonymous">
      Non dir� mai il mio nome
    </a>

Per evitare questo problema, nel creare collegamenti ipertestuali con le azioni dell'applicazione si dovrebbe sempre usare l'helper `link_to()`. Quando si vuole generare unicamente la parte dell'URL senza l'HTML, allora si pu� utilizzare l'helper `url_for()`.

Un helper � una funzione PHP definita da symfony con lo scopo di essere usata all'interno dei template. Restituisce del codice HTML ed � pi� veloce da usare rispetto a scrivere il codice HTML a mano. Il Listato 4-9 mostra l'uso degli helper per i collegamenti ipertestuali.

Listato 4-9 - Gli helper `link_to()` e `url_for()`

    [php]
    <p>Buongiorno, mondo!</p>
    <?php if ($hour >= 18): ?>
      <p>O dovrei dire buona sera? Sono gi� le <?php echo $hour ?>.</p>
    <?php endif; ?>
    <form method="post" action="<?php echo url_for('contenuto/update') ?>">
      <label for="name">Qual'� il tuo nome?</label>
      <input type="text" name="name" id="nome" value="" />
      <input type="submit" value="Ok" />
      <?php echo link_to('Non dir� mai il mio nome', 'contenuto/update?nome=anonymous') ?>
    </form>

L'HTML risultante sar� lo stesso del precedente, eccetto che quando si cambiano le regole delle rotte, tutti i template si comporteranno correttamente e riformatteranno gli URL di conseguenza.

L'utilizzo dei form merita uncapitolo a s� stante, dato che symfony fornisce molti strumenti per renderli ancora pi� facili. Si imparer� di pi� su questi helper nel Capitolo 10.

L'helper `link_to()`, cos� come altri helper, accetta un altro argomento per opzioni spaciali e addizzionali attributi per i tag. Il Listato 4-10 mostra un esempio di argomento opzione e dell'HTML risultante. L'argomento opzione pu� essere sia un array associativo che una semplice stringa e mostrer� una coppie `chiave=valore` separate da spazi.

Listato 4-10 - Molti helper accettano un argomento opzione

    [php]
    // L'argomento opzione come array associativo
    <?php echo link_to('Non dir� mai il mio nome', 'contenuto/update?nome=anonymous',
      array(
        'class'    => 'special_link',
        'confirm'  => 'Sei sicuro?',
        'absolute' => true
    )) ?>

    // L'argomento opzione come stringa
    <?php echo link_to('Non dir� mai il mio nome', 'contenuto/update?nome=anonymous',
      'class=special_link confirm=Sei sicuro? absolute=true') ?>

    // Entrambi i modi hanno generano lo stesso risultato
     => <a class="special_link" onclick="return confirm('Sei sicuro?');"
        href="http://localhost/frontend_dev.php/contenuto/update/nome/anonymous">
        Non dir� mai il mio nome</a>

Ogni volta che si utilizza un helper di symfony che genera un tag HTML, � possibile inserire ulteriori attributi al tag (come l'attributo `class` nell'esempio del Listato 4-10) nell'argomento opzione. Questi attributi si possono anche scrivere nel modo "sporco e veloce" dell'HTML 4.0 (senza i doppi apici) e symfony li mostrer� con una formattazione XHTML. Questa � un'altra ragione per cui gli helper sono pi� veloci da scrivere rispetto all'HTML.

>**NOTE**
>Essendo che � necessaria una ulteriore analisi e trasformazione la sintassi con stringa � un po' pi� lenta della sintassi con gli array.

Come tutti gli helper di symfony, gli helper per i link sono numerosi e hanno molte opzioni. Il Capitolo 9 li descrive in dettaglio.

Ottenere informazioni dalla Request
-----------------------------------

Se l'utente invia informazioni tramite un form (di solito in una richiesta POST) o tramite l'URL (richiesta GET), � possibile recuperare i dati relativi dall'azione con il metodo `getParameter()` dell'oggetto `sfRequest`. Il Listato 4-11 mostra come in `update`, si recupera il valore del parametro `nome`.

Listato 4-11 - Recuperari i dati dal parametro Request dell'azione

    [php]
    <?php

    class contenutoActions extends sfActions
    {
      // ...

      public function executeUpdate($request)
      {
        $this->nome = $request->getParameter('nome');
      }
    }

Per convenienza, tutti i metodi `executeXxx()` prendono come primo argomento l'oggetto corrente `sfRequest`.

Se l'elaborazione dei dati � semplice, non si avr� nemmeno bisogno di usare l'azione per recuperare i parametri della richiesta. Il template ha accesso a un oggetto chiamato $sf_params`, che offre un  metodo ``get`() per recuperare i parametri di richiesta, proprio come il `getParameter()` nell'azione.

Se `executeUpdate()` fossero vuoti, il Listato 4-12 mostra come il template `updateSuccess.php`  dovrebbe recuperare lo stesso parametro `nome`.

Listato 4-12 - Recuperare i dati dei parametri della Request direttamente nel template

    [php]
    <p>Ciao, <?php echo $sf_params->get('nome') ?>!</p>

>**NOTE**
>Perch� non usare invece le variabili `$_POST`, `$_GET`, o `$_REQUEST`? Perch� allora l'URL verr� formattato in modo diverso (come in `http://localhost/articles/europe/france/finance.html`, senza `?` n� `=`), le normali variabili PHP non funzionano pi� e solo il sistema delle rotte sar� in grado di recuperare i parametri di richiesta. E si potrebbe voler aggiungere un filtro in ingresso per impedire l'iniezione di codice maligno, che � possibile solo se si mantengono tutti i parametri di richiesta in un solo contenitore di parametri.

L'oggetto $sf_params` � pi� potente di un semplice getter equivalente a un array. Ad esempio, se si vuole fare il test dell'esistenza di un parametro di richiesta, si pu� semplicemente usare il metodo $sf_params->has()` invece di verificare l'effettivo valore con `get()`, come nel Listato 4-13.

Listato 4-13 - Fare il test dell'esistenza di un parametro richiesta nel template

    [php]
    <?php if ($sf_params->has('nome')): ?>
      <p>Buongiorno, <?php echo $sf_params->get('nome') ?>!</p>
    <?php else: ?>
      <p>Buongiorno, John Doe!</p>
    <?php endif; ?>

Si potrebbe avere gi� capito che questo pu� essere scritto in una sola riga. Come con la maggior parte dei metodi get di symfony, sia il metodo `$request->getParameter()` nella azione che il metodo `$sf_params->get()` nel template (che, di fatto, chiama lo stesso metodo sull'oggetto stesso) accettano un secondo argomento: il valore predefinito da utilizzare se il parametro richiesta non � presente.

    [php]
    <p>Buongiorno, <?php echo $sf_params->get('nome', 'John Doe') ?>!</p>

Riepilogo
---------

In symfony, le pagine sono formate da una azione (un metodo nel file `actions/actions.class.php` file prefissato con `execute`) e un template (un file nella cartella `templates/`, che generalmente termina con `Success.php`). Queste sono raggruppate in moduli, in base alla loro funzione nell'applicazione. La scrittura dei template � facilitata dagli helper: sono funzioni fornite da symfony che restituiscono codice HTML. Ed � necessario pensare alla URL come ad una parte della risposta, che pu� essere formattata in base alle proprie necessit�, pproprio per questo si dovrebbe evitare di utilizzare qualsiasi riferimento diretto alla URL nei nomi delle azioni o nel recupero dei parametri della richiesta.

Una volta che questi principi di base vengono compresi, si pu� gi� procedere con la scrittura di una completa applicazione web con symfony. Ma sarebbe un lavoro pi� lungo di quanto dovrebbe, dal momento che quasi tutte le attivit� che si dovranno realizzare nel corso dello sviluppo dell'applicazione vengono facilitate in un modo o nell'altro da qualche caratteristica peculiare di symfony ... ed � per questo che il libro non si ferma qua.
