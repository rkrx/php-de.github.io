---
layout: guide
permalink: /jumpto/di/
root: ../..
title: "Dependency injection"
creator: rkr
group: "Objektorientierte Programmierung (OOP)"
orderId: 5

author:
    -   name: rkr
        profile: 35991

inhalt:
    -   name: "Dependency injection"
        anchor: di
        simple: "Dependency injection"
    -   name: "Dependency injection container"
        anchor: dic
        simple: "Dependency injection container"
---

## Einleitung

Seit PHP4 steht Entwicklern die Möglichkeit offen, Programme objektorientert zu entwickeln. Mit der Einführung von PHP5 wurde diese Möglichkeit aber erst richtig interessant, da zuvor Objekte nur als Kopie weitergegeben werden konnten und seit PHP5 standardmäßig referenziert übergeben werden.
 
Objektorientierung stellt eine Möglichkeit dar, ein Programm in Dinge zu zerlegen und so das Modell einer Software für Menschen leichter zugänglich zu machen. Menschen können sich in der Regel besser in Situationen reinversetzen, wenn sie in Bildern denken können und einzelne Aufgaben konkreten Dingen zuordnen können, die diese Aufgaben bearbeiten.

Besonders gut funktioniert Objektorientierung dann, wenn man komplexe Arbeitsschritte so sehr vereinfacht, dass jedes Objekt nur noch eine Aufgabe bearbeitet, die sich nicht mehr weiter vereinfachen lässt und nicht mehr als diese simple Aufgabe lösen kann. Selten sind Aufgaben so einfach. Daher gibt es auf einer höheren Ebene Objekte, deren einzige Aufgabe es ist, aus komplexeren Aufgaben weniger komplexe Aufgaben zu extrahieren und diese dann an Objekte zu delegieren, die diese Aufgaben dann verarbeiten, oder ihrerseits wieder vereinfachen und weiterreichen. Das kann theoretisch über viele Schichten gehen und hängt von der eigentlichen Komplexität der Aufgabe ab.

*Man muss auch nicht zwingend diese Abstufung einhalten. Wenn es die Aufgabe nicht erfordert, dann ist es legitim, ein Objekt auf unterer Ebene mehr als einen Schritt machen zu lassen. Wichtig ist, dass man sich offen hält, dieses Objekt auch wieder aufteilen zu können, wenn sich ändernde Bedingungen dies erforderlich machen.*

Damit das funktionieren kann, müssen Objekte aber voneinander wissen. Objekte wissen immer nur, mit wem sie direkt zu tun haben, aber nicht direkt, wer das Objekt war, von dem sie eine Aufgabe bekommen haben. Auch dann nicht, wenn es als Parameter mitgegeben wurde, da nur das Interface dieses Objekts sichtbar ist und es so auch eine beliebige Implementation mit gleichem Interface sein kann. Diese Ausgangssituation ist wichtig, denn nur so lässt sich eine Anwendung entwickeln, bei der sich Teile später ohne viel Aufwand austauschen lassen.

Jetzt gibt es zwei Wege, wie Objekte voneinander wissen können. Der eine lautet *Dependency Injection* und wird in diesem Artikel vorgestellt. Der Andere lautet, dass man die Instanz der Delegaten entweder direkt in der Klasse erzeugt, oder über einen globale(re)n Kontext (beispielsweise einen Service-Locator) angefordert.

## Dependency-Injection

### Was ist DependencyInjection und welches Problem wird damit gelöst?

Versuchen wir mal eine künstliche Situation zu zeichnen, mit der auch Einsteiger einen Faden haben, an dem sie sich in die Situation hineinversetzen können.

Wir wollen ein Haus bauen. Das **Projekt** *Hausbau* ist hier für den Softwareentwickler die **Klasse**.

Nehmen wir an, wir (also ich; also eine Person) wollen alles selbst machen. Kein Architekt, kein Statiker, kein Maurer, kein Elektriker und kein Raumausstatter. Heißt, wie machen es nicht objektorientiert, oder für einen Softwareentwickler ausgedrückt: Wir arbeiten mit Funktionen; Prozedural. Oder noch anders ausgedrückt: Unsere Applikation ist ein großes Objekt, welches ganz viele Zuständigkeiten hat. Jemand der es schafft auf diese Weise ein Haus zu bauen, oder eine komplexere Applikation zu erstellen, verdient in jedem Fall eine Menge Respekt.

Dann gibt es viele, die zwar nichts so richtig an andere Firmen abgeben wollen, aber trotzdem nicht alles selbst machen möchten. Diese Bauherren geben dann ihre Aufgaben an Freunde und Verwandte oder vielleicht gerade erst neu gewonnene Nachbarn weiter. Jeder hat ja einen Architekten, einen Statiker, einen Elektriker und einen Maurer in der Familie oder zumindest im Bekanntenkreis. Nur als Raumausstatter ist man selbst eben unersetzbar. Schlecht wird es dann, wenn man feststellt, dass sich jemand für die im zugeteilte Arbeit eigentlich überhaupt nicht eignet. Dann muss Ersatz her. Wieder aus dem Bekanntenkreis. Notfalls wird jemand angelernt, der es bislang auch noch nicht konnte. Das ist für den Softwareentwickler dann die Situation mit der direkten Instanziierung oder der harten Verdrahtung durch eine Instanz aus (z.B.) einem Service-Locator innerhalb einer Klasse. Das lässt sich alles nicht mehr so leicht ändern, wenn man erst mal mitten drin ist.

Dann gibt es doch auch einige, welche die eigentliche Arbeit des Hausbaus von Fachleuten erledigen lassen. Also beschäftigen sie einen Architekten, einen Statiker, einen Maurer, einen Elektriker und einen Raumausstatter aus dem Telefonbuch. Alles, was man selbst machen muss ist, die jeweiligen Qualifikationen zu prüfen und den eigentlichen Aufgaben zu definieren. Wir sind weder emotional, noch auf Grund der eigentlichen Qualifikation an den jeweiligen Fachmann gebunden. Wir können mittendrin feststellen, dass die ausstehende Arbeit von einem anderen Fachmann besser erledigt werden kann und müssen das Modell unseres Projekts (also die Klasse) nicht anpassen, um diesen Wechsel vorzunehmen. Wir geben diesen Wechsel einfach von außen vor.
 
Es ist nicht immer ganz einfach, die reale Welt mit Mitteln der Softwareentwicklung zu beschreiben. Jede Analogie hat immer auch einen leichten Beigeschmack. Das oben skizzierte Beispiel würde übertragen auf die Softwarewelt folgendes beschreiben:

Ich habe eine Klasse, die ihrerseits Aufgaben an andere Klassen weiterverteilen möchte. Ich kommuniziere mit anderen Klassen immer nur über ein Interface, also mit öffentlich zugänglichen Eigenschaften (Properties) oder Methoden. Das heißt, dass folgende Figuren im Grunde die gleiche Folge haben:

~~~ php

    class CopyFiles {
        /**
         * @param array $filepaths
         */
        public function copyFilesTo(array $filepaths) {
            $res = FtpFileSystem::connect($GLOBALS['FTP_OPTIONS']);
            foreach($filepaths as $filepath) {
                FtpFileSystem::copy($res = $filepaths, basename($filepath));
            }
            FtpFileSystem::close($res);
        }
    }
    
~~~

~~~ php

    class CopyFiles {
        /** @var FtpFileSystem */
        private $filesystem = null;
        
        /**
         */
        public function __construct() {
            $this->filesystem = new FtpFileSystem($GLOBALS['FTP_OPTIONS']);
        }
        
        /**
         * @param array $filepaths
         */
        public function copyFilesTo(array $filepaths) {
            foreach($filepaths as $filepath) {
                $this->filesystem->copy($filepaths, basename($filepath));
            }
        }
    }
    
~~~

~~~ php

    class CopyFiles {
        /** @var FileSystem */
        private $filesystem = null;
        
        /**
         */
        public function __construct(FileSystem $filesystem) {
            $this->filesystem = $filesystem;
        }
        
        /**
         * @param array $filepaths
         */
        public function copyFilesTo(array $filepaths) {
            foreach($filepaths as $filepath) {
                $this->filesystem->copy($filepaths, basename($filepath));
            }
        }
    }
    
~~~

Figur #3 hat gegenüber der anderen Figuren einen entscheidenden Vorteil: Ich kann ein beliebiges *FileSystem* vorgeben, denn *CopyFiles* muss nicht wissen, wohin genau oder wie die Dateien kopiert werden sollen. *CopyFiles* weiß nur, welche Dateien kopiert werden sollen. Dadurch wird aus einem *CopyFiles*-Objekt, welches nur Dateien auf einen FTP-Server spielen kann, eine Schablone, die das Konzept *"Kopiere mir Dateien auf ein von mir vorgegebenes Dateisystem"* implementiert und wird dadurch wesentlich universeller einsetzbar:
 
~~~ php

    $files = array(/* ... */);
    $filesystem = new SshFileSystem(/* ... */);
    $fileCopier = new CopyFiles($filesystem);
    $fileCopier->copy($files);

~~~
 
~~~ php

    $files = array(/* ... */);
    $filesystem = new LocalFileSystem(/* ... */);
    $fileCopier = new CopyFiles($filesystem);
    $fileCopier->copy($files);

~~~
 
~~~ php

    $files = array(/* ... */);
    $filesystem = new WebDavFileSystem(/* ... */);
    $fileCopier = new CopyFiles($filesystem);
    $fileCopier->copy($files);

~~~
 
~~~ php

    $files = array(/* ... */);
    $filesystem = new DropboxFileSystem(/* ... */);
    $fileCopier = new CopyFiles($filesystem);
    $fileCopier->copy($files);

~~~

*Bei einem Dateisystem ist das jetzt einfach gewesen. Und so wahnsinnig viele Fälle gibt es nicht, wo man Dependency-Injection sinnvoll anwenden kann. Ok gelernt, und weiter?*
Tatsächlich ist es genau umgekehrt. Es gibt nur ein paar wenige Fälle, bei denen man _nicht_ sinnvoll mit Dependency-Injection modellieren kann. Es kann in einzelnen Fällen auch durchaus sein, dass die Applikation besser nur aus direkten Abhängigkeiten besteht. Aber solche Szenarien sind verrückt selten.

Aber lasst und den Fall mal andersherum skizzieren. Jetzt haben wir DI und stellen die Frage, ob wir das wirklich brauchen.

* Welche konkreten Nachteile ergeben sich durch direkte Abhängigkeiten?
* Ich habe eine Applikation, in der alles größtenteils mit direkten Abhängigkeiten gelöst wurde, soll ich das jetzt alles ändern?

### Was sind Klassenmethoden (statische Methoden) und warum sollte man diese vermeiden?

## Depedency-Injection-Container

### Was ist eigentlich ein DependencyInjectionContainer (DIC)?

Ein guter DI-Container erfüllt folgende Eigenschaften:

* Er fördert Inversion-of-Control
* Er ist praktisch unsichtbar
* Er benötigt nur so viel Konfiguration (bzw. "Einmischung"), wie unbedingt notwending
* Er unterstützt Refactoring-Tools der großen IDEs so gut es geht

Was ein DIC alles kann, kann man vielleicht am Beispiel von zwei unterschiedlichen Implementierungen erklären: [PHP-DI](http://php-di.org/) und [Pimple](http://pimple.sensiolabs.org/).

Was ist PHP-DI eigentlich? PHP-DI ist ein Dependency-Container, der neben allem, was der Symfony-DIC (ohne Extension) und Pimple können, auch noch Autowiring unterstützt. Er erlaubt also, dass man sich eine Instanz einer Klasse holen kann, ohne diese in vielen Fällen vorher in der Konfiguration des DIC bedacht zu haben. PHP-DI macht dann - sofern nicht anders konfiguriert - ein Singleton* aus dieser Klasse. Das Singleton ist dann im Kontext des Containers einzigartig. Heißt, wenn du einen zweiten Container instanziierst, dann wird für diesen wieder eine neue Instanz erzeugt. Dann kann PHP-DI noch Lazy-Injection (was du, wenn du streng nach SoC/SRP arbeitest, eher selten brauchen wirst) und als Object-Factory benutzt werden, was streng genommen nicht direkt mit einem DIC zu tun hat, aber eine wichtige Eigenschaft ist.

Pimple ist ein aufgebohrtes Array, dass man theoretisch auch durch eine Klasse ersetzen kann:

~~~ php

    class MyDIC {
        use InstanceCache;
    
        /** @return LoggerInterface */
        public function getLogger() {
            return $this->cache(__FUNCTION__, function () {
                $logger = new Monolog\Logger();
                /* ... */
                return $logger;
            });
        }
    
        /** @return MyController */
        public function getMyController() {
            return $this->cache(__FUNCTION__, function () {
                $logger = $this->getLogger();
                $myService = $this->getMyService();
                return new MyController($logger, $myService);
            });
        }
    
        /** @return MyService */
        public function getMyService() {
            return $this->cache(__FUNCTION__, function () {
                return $this->cache(__FUNCTION__, function () {
                $logger = $this->getLogger();
                return new MyService($logger);
            });
        }
    }
    
    $dic = new MyDIC();
    $service = $dic->getMyController();
  
~~~

Pimple ist da etwas flexibler, unterstützt aber kein Typehinting, was die Frage erschwert, wo eine Klasse in einem Projekt überall eingesetzt wird. Klar kann man überall PhpDoc-Definitions einbauen ... da wird aber aus Erfahrung selten jemand diszipliniert genug sein, das auch durchzuziehen.

Man wird im Falle von PHP-DI folgenden Boilerplate-Code brauchen, damit es den Container überhaupt gibt:

~~~ php

    $builder = new ContainerBuilder();
    $builder->addDefinitions(__DIR__.'/config/di.php');
    $container = $builder->build();
    $container->useAnnotations(false);  
  
~~~

Jetzt haben wir einen Container. Die Config ist leer. Keine Annotations. Soweit würde man das mit Pimple auch noch hinbekommen.

~~~ php

    $myClass = $container->get(MyClass::class); // ::class ist ein Feature, dass erst ab PHP5.5 verfügbar ist.
  
~~~

Ich bekomme tatsächlich eine Singleton-Instanz von MyClass, ohne, dass ich dies irgendwo konfiguriert habe. MyClass darf dafür in diesem Falle allerdings kein Interface und keine abstrakte Klasse sein. 

Nun hat die MyClass-Klasse ihrerseits wieder Abhängigkeiten, die ich eigentlich via Constructor-Injection definieren müsste:

~~~ php

    <?php
    namespace Test;
    
    use Psr\Log\LoggerInterface;
    
    class MyClass {
        /** @var LoggerInterface */
        private $logger;
        /** @var SomeService */
        private $service;
    
        /**
         * MyClass constructor.
         *
         * @param LoggerInterface $logger
         * @param SomeService $service
         */
        public function __construct(LoggerInterface $logger, SomeService $service) {
            $this->logger = $logger;
            $this->service = $service;
        }
    
        /**
         * @param string $workload
         * @return bool
         */
        public function doSomething($workload) {
            $this->logger->info(sprintf("Got some work to do: %s", $workload));
            $convertedData = $this->service->convert($workload);
            $result = $this->service->process($convertedData);
            $this->logger->info(sprintf("Work is ready. Result was: %s", json_encode($result)));
            return $result;
        }
    }
    
    /* ... */
    
    class SomeService {
        /**
         * @param string $workload
         * @return array
         */
        public function convert($workload) {
            /* ... */
            return $workload;
        }
    
        /**
         * @param array $workload
         * @return bool
         */
        public function process(array $workload) {
            /* ... */
            return $result;
        }
    }
    
~~~

Solange diese Abhängigkeiten keine Interfaces, abstrakten Klassen oder primitiven Datentypen im *Constructor* erwarten, oder via Method-Injection bedacht werden müssen, wird PHP-DI alles automatisch auflösen. Andernfalls muss man die Konfiguration von PHP-DI aufsuchen und genau spezifizieren, was genau ein ```LoggerInterface``` eigentlich ist:

~~~ php

    // Config
    use function DI\factory;
    use Psr\Log\LoggerInterface;
    
    return [
        LoggerInterface::class => factory(function () {
            /* ... */
            return $logger;
        }),
    ];

~~~

Also hat man eine zentrale Stelle, wo man definiert, dass beispielsweise ein Monolog für den Einsatz als Logger bereitgestellt werden soll. Sollte ich irgendwann die Methodensignator meiner Construct-Methoden ändern, wird PHP-DI sich um den Rest kümmern.

Nachher fordere ich meine Klassen über den ```Ctor``` in etwa so an, wie ich vorher einen Autoloader via ```new``` beschäftigt hätte. Meine Klassen sind absolut sauber. Keine Magie - alles was hier passiert, kann ebenso von Hand nachgebaut werden. Ich kann bald jede Serviceklasse einfach über ein neues CMD-Script einbinden und aufrufen, wenn ich auf diese Weise irgendwas fixen kann:

~~~ php

    $container->call(function (OrderStatusService $statusService, OrderRepository $repository, TrackingService $trackingService) {
        $order = $repository->get(1234567);
        $statusService->markShipped($order);
        $trackingService->addReference($order, '206792171027');
    });  

~~~

Völlig egal, wie komplex die Erzeugung der abhängigen Klassen ist. Während ich mit PHP-DI nachher eine Config-Datei habe, die zusammen genommen 500 Zeilen lang ist, habe ich in Pimple mindestens das doppelte an Zeilen - optimistisch geschätzt.

Wenn ich da jetzt Klassen habe, die ihrerseits Abhängigkeiten haben, die nicht in jedem Fall zum Zuge kommen, dann kann auch via PHP-DI diese Abhängigkeiten in einen Lazy-Proxy packen, der erst dann die dahinterliegende Klasse aufruft, wenn sie gebraucht wird.

Ähnlich wie bei Pimple und jedem anderen DI-Container wird das Ergebnis sein, dass der DIC in deinen Klassen nicht in Form eines Service-Locator auftaucht, sondern alle Abhängigkeiten zum Zeitpunkt der Erzeugung der jeweiligen Klasse bereits instantiiert wurden. Bei Pimple wird das oft vernachlässigt, weil man eben für jede Klasse eine Konfiguration bereitstellen muss. Bei Symfony ist das (meine ich) heute auch noch so. Das ist der Unterschied zwischen (Vorsicht, Analogie) *require/include* und *autoloading*.

Und dann taucht PHP-DI bei mir gerne doch noch mal auf. In Factories:

~~~ php

    use DI\Container;
    
    class SomeFactory {
        /** @var Container */
        private $container;
    
        /**
         * @param Container $container
         */
        public function __construct(Container $container) {
            $this->container = $container;
        }
    
        /**
         * @return Entity
         */
        public function createEntity($name, $arguments) {
            return $this->container->make(Entity::class, ['name' => $name, 'arguments' => $arguments]);
        }
    }
    
~~~

Wenn meine Entities also selbst wieder irgendwelche Abhängigkeiten haben (oder auch nicht)... Ich werde mich in der Factory nicht direkt drum kümmern müssen. Würde es die Argumente ```$name``` und ```$arguments``` nicht geben, dann könnte man solche Fabriken theoretisch auch durch Generics standardisieren. Das wäre etwas, was in künftigen PHP-Versionen dann wieder viel Code einsparen könnte:

~~~ php

    use DI\Container;
    
    class Factory<Entity> {
        private Container $container;
    
        function __construct(Container $container) {
            $this->container = $container;
        }
    
        public function create(): Entity {
            return $this->container->make(Entity::class);
        }
    }  

~~~

Das könnte man zwar heute schon. Aber dann ohne IDE-gestützte Nachvollziehbarkeit.

Was hat das in meinen Projekten aber nun tatsächlich gebraucht? Tatsächlich ist es doch nur ein anderer DIC. Kann man auch alles mit Pimple machen.

* Wir Entwickler sind faul. Wenn es einfacher ist und schneller geht, einen komplexen Parameter als String zu übergeben, anstatt eine Klasse dafür zu bauen, dann machen wir das. Wird der Parameter irgendwann komplexer (oder auch nicht), dann kann man sich diesem Problem immer noch annehmen. Für mich gab es da einen deutlichen Shift. Die Bereitschaft neue Klassen für angemessene Situationen zu erstellen fühlt sich nicht mehr wie over-engineering an. Selbst, wenn ich eine Zahl habe, die mein aktuelles Environment beschreibt (beispielsweise die ID des System-Mandanten), dann pack ich die in eine Klasse und kann diese Klasse überall komfortabel dank Autowiring einlinken.
* Meine Klassen sind schlanker geworden. Alle sprechen von SoC und dem SRP. Es gehört aber auch eine Menge Disziplin dazu, alles immer richtig zu machen. Besonders, wenn man in kurzen Zyklen liefern will.
* Ich bin wesentlich flexibler was einen Switch innerhalb einer Applikation auf neuere Komponenten betrifft. Da ich also tendenziell alles sauberer schreibe, kann ich leichter einzelne Interfaces durch neuere Implementationen bespielen und so alte, historisch gewachsene Applikationen nach und nach erneuern.
* Meine Applikationen sind um ein vielfaches Zugänglicher und Testbarer geworden. Der Unterschied ist gravierend. Und ich kenne keine Applikation, die in dieser Hinsicht so sauber umgesetzt wurde, dass mich jetzt mal hinstelle und sage: Niemand wird mir eine PHP-basierte Applikation mit mehr als 100k loc zeigen können, die das auch ohne die von mir beschriebenen Techniken hinbekommt.
* Ich mache wirklich ungerne Werbung für andere, aber ich nutze PHPStorm. Das ist die einzige IDE, die aktuell in der Lage ist, meinen PHP-Code statisch zu analysieren und mit zu sagen, was wo wie genutzt wird. PHPStorm liebt PHP-DI.

Ich habe davor erst diese klassen-basierte DIC-Methode von weiter oben verwendet. Danach den DIC von Zend. Hat funktioniert, war aber alles andere als flexibel. Ich will nicht sagen, dass PHP-DI alle Probleme unserer modernen Gesellschaft löst, aber es hat mir gezeigt, wie viel Potential noch für die Entwicklung von Applikationen freigesetzt werden kann.

### Was ist der Unterschied zwischen einen DIC und einem ServiceLocator?

Ein Service-Locator wirkt innerhalb einer Klasse. Ein DIC wirkt von außerhalb einer Klasse. Vielleicht kann man dies mit einem Codebeispiel erklären:

Service-Locator:

~~~ php

    class MyClass {
        /**
         */
        public function someMethod() {
            $dep = SL::get(MyDependency::class);
            $dep->doSomething();
        }
    }

~~~

Depndency-Injection(-Container):

~~~ php

    class MyClass {
        /** @var MyDependency */
        private $dep;
    
        /**
         * @param MyDependency $dep
         */
        public function __construct(MyDependency $dep) {
            $this->dep = $dep;
        }
    
        /**
         */
        public function someMethod() {
            $this->dep->doSomething();
        }
    }

~~~

Bei der zweiten Figur wird schnell klar: Ob hier ein DIC gewirkt hat, oder die Instanz *manuell* erzeugt wurde, ist irrelevant. Wenn ich einen automatisierten Test schreibe, dann habe ich bei der zweiten Methode kaum Probleme, eine gefakte (gemockte) Abhängigkeit zu injizieren. Bei der Variante mit dem Service-Locator hätte ich hier eine weit höhere Hürde. Das würde auch zutreffen, wenn ich den Service-Locator als Instanz über den *Constructor* oder über eine *Methode* injiziere.

### Welche Interfaces/Contracts gibt es?

* [container-interop/container-interop](https://github.com/container-interop/container-interop)
* [rkr/php-ioc-contract](https://packagist.org/packages/rkr/php-ioc-contract)

### Welche DIC-Implementationen gibt es und was können diese?

|Funktion/DIC|PHP-DI|Pimple|Symfony DI|Zend DI|Aura DI|
|----|----|----|----|----|----|
|Aktuelle Version|~4|~3|?|?|
|Configuration|PHP, Annotations|PHP|PHP, Yaml, XML|PHP|PHP|
|Autowiring|Ja|Nein|Nein|Nein|Nein|
|Lazy-Instantiation|Ja|Nein|Nein|Nein|Nein|
|Object-Factory|Ja|Nein|Nein|Nein|Nein|
|Method-Invokation|Ja|Nein|?|?|?|

