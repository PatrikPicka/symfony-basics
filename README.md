# Symfony basics (scripts and tutorials)

CREATING NEW SYMFONY PROJECT (Commands)
 - symfony new path/to/dir/name --version="^your.version"
 - composer require symfony/maker-bundle
 - composer require doctrine
 - composer require doctrine/annotations

CREATING TEMPLATES AND INSTALLING ASSETS
 - composer require twig
 - composer require symfony/webpack-encore-bundle
 - npm install

INSTALLING SASS, BOOTSTRAP, jQuery
 - npm install node-sass
 - npm install sass-loader
	-- webpack.config.js - odkomentovat řádek s ".enableSassLoader()"
 - npm install bootstrap --save-dev
	- assets/app.js - přidat kód "require("bootstrap");"
	- assets/styles/app.scss - přidat kód "@import '~bootstrap/scss/bootstrap.scss';"
 - npm install jquery
	- assets/app.js - přidat kód "const \$ = require("jquery"); global.\$ = global.jquery = \$;"

SYMFONY PROFILER / DEBUGING(bottom info bar)
 - composer require --dev symfony/web-profiler-bundle
 - composer require --dev symfony/var-dumper (for using dd(your_data_to_dump);)
 - composer require --dev symfony/debug-bundle 
  	- (for dumping informations into the profiler so you do not have to use dd(your_data_to_dump) but instead you can use dump(your_data_to_dump))

DATA FIXTURES (for creating test data etc.)
 - composer require --dev orm-fixtures
 - bin/console make:fixtures --- creates fixture
 - bin/console doctrine:fixtures:load --- executes fixtures

AUTHENTICATION AND SECURITY
 - composer require symfony/orm-pack
 - composer require symfony/form
 - composer require symfony/security-bundle
 - composer require symfony/validator 

LIST OF COMMANDS 
 - php bin/console list

CREATING CONTROLLER FROM CLI
 - php bin/console make:controller
CREATING DB
 - php bin/console doctrine:database:create
CREATING ENTITY
 - php bin/console make:entity


RESET DB SCRIPT
 - bin/console doctrine:schema:drop --force && rm ./migrations/*.php && bin/console doctrine:schema:update --force && bin/console doctrine:migrations:diff && bin/console doctrine:migrations:migrate


CONTROLLERS
 - generating route ---> $this->generateUrl("name_of_the_route", ["param_1" => 10, "param_2" => 20], UrlGeneratorInterface::ABSOLUTE_URL);
	- UrlGeneratorInterface::ABSOLUTE_URL - with this option it will generate absolute URL with http(s)://your_url.end/......


CREATING COOKIE
 - $cookie = new Cookie("my_new_cookie", "cookie_value", time() + (2 * 365 * 24 * 60 * 60));
 - $res = new Response();
 - $res->headers->setCookie($cookie);
 - $res->send();

DELETING COOKIE
 - $res = new Response();
 - $res->headers->clearCookie("my_new_cookie");
 - $res->send();


SESSIONS
SessionInterface object (DI)
 - set a session value ---> $session->set("name", "session_value");
 - check if session has ---> $session->has("name");
 - get value from session ---> $session->get("name")
	- For more methods look into the SessionInterface object


ROUTES
 - @Route("/user/{id?}/{page}", name="your_route_name", requirements={"page"="\d+"}, defaults={"page"="dashboard"})
	 - {id?} ---> ? means that the parameter is optional
	 - name="your_route_name" ---> Pojmenování routu pomocí kterého pak můžeme volat při $this->redirect("your_route_name");
	 - requirements={"page"="\d+"} ---> specifikace zadávaného parametru (parametr musí být číslo)
	 - defaults={"page"="dashboard"} ---> sets the default value for parameter in url


SERVICES
 - You can't call other services in any function of service you can call other services only in the __construct magic method
 - If you want to create another function which will autorun same as the __construct method you have to use annotations
	"/**
	  * @required
	  */"
	- This will tell symfony that this function should run automatically

ADD A PARAMETER TO SERVICE
 - services.yaml ->
	Namespace\Of\The\Services
		arguments:
			$parameter: "your param value" || "@App\Your\Second\Service" (you can use this to pass another service as parameter to your service)


SPECIFY PROPERTY (variable value) IN SERVICE YAML FILE
 - You can define properties inside your service inside services.yaml 
	 - Namespace\Of\Your\Service
		 - properties:
			 - your_name_of_prop: "@Namespace\To\Another\Service" || "your_value"
	- then you have to have a public variable inside your service: public $your_name_of_prop; -- and symfony will automatically set these vars
			- But you wont be able to access them inside __construct method  



ADD SPECIFIC SERVICE TO CONTROLLER
 - services.yaml -> 
 - Namespace\Of\The\controller
  	- bind:
    		- $your_variable: '@service' (@monolog.logger.doctrine)
 - --> Inside of the controller you have to specify this in magic method __construct($your_variable)


LAZY OPTION FOR SERVICES
 - You can specify lazy option for service inside services.yaml file => service wont be loaded until u specificly use it
 - composer require symfony/proxy-manager-bridge --- this is needed for lazy option to work


ASSIGN ALIAS TO SERVICE
 - inside services.yaml ->
 - app.your_name
	- class: Namespace\Of\Your\Service
	 	- ---You can assign arugments etc---
		- arguments:
			- $name_of_arg: "value of arg"
- Namespace\Of\Your\Service: "@app.your_name"


TAGS FOR SERVICES
 - You can specify a special tag for a service by using "tags:" in services.yaml
 - Then you have to specify which tag is the service bound to -> 
	- {name: doctrine.event_listener, event: postFlush} --- this will run after flush is executed (you have to create the sam method as the event "postFlush()")
	- --- You can find the tags in Symfony docs

DOWNLOADING A FILE FROM SERVER
 - services.yaml ---> parameters: -> download_directory: "../public/downloads"
 - inside controller ---> $path = $this->getParameter("download_directory"); --- This will get the path to download folder
	- -> return $this->file($path . "your_file"); --- This will download the file


PARAM CONVERTER
 - if you want to use param converter (Route("url.../{id_user}") ---> function (User(entity) $id_user){$id_user->getName();}
 	- --- You have to install sensio/framework-extra-bundle ---> composer require sensio/framework-extra-bundle



DOCTRINE (CREATE A CUSTOM QUERY)
 - $conn = $em->getConnection(); ($em = EntityManagerInterface)
 - $sql = "SELECT * FROM users as  u WHERE u.id = :id AND u.username = :username;";
 - $stmt = $conn->prepare($sql);
 - $data = $stmt->executeQuery(["id" => 1, "username" => "Patrik Picka"]);
 - dd($data->fetchAll());


CASCADE DELETING AND PERSISTING IN RELATIONS
 - @ORM\OneToMany(targetEntity=Video::class, mappedBy="var_in_relation_entity", cascade={"ALL"} --- 
 - @ORM\ManyToOne(targetEntity=User::class, inversedBy="videos", onDelete="CASCADE")
	- --- orphanRemoval = When you call remove on the object you want to delete thru the parent entity this will delete it if you dont use this the record will stay in the DB and could be reused
 - @ORM\OneToOne(targetEntity=Address::class, cascade={"persist", "remove"}) ---> if you have cascade-persist then when you create new address and add it to user you have to persist just a User not the address


LAZY LOADING / EAGER LOADING
 - Normally doctrine/symfony gets only emty array of related objects and loads it only when you want to use them = Lazy loading
 - If you want to load all the related object u have to write your own function inside repository and use query builder
	- --- public function findAllVideos($id) : ?User { return $this->createQueryBuilder("u")->innerJoin("u.videos", "v")->addselect("v")->andWhere("u.id = :id")->setParameter("id", $id)->getQuery()->getOneOrNullResult();}


POLYMORPHIC TYPE OF RELATIONS
 - Create one abstract class for example - File - this class will have relation with Author ManyToOne
 - Then create two classes for example - Movie, Pdf - these classes will extend File class
	- Then you have to specify on the abstract parent class these things
	 * @ORM\InheritanceType("SINGLE_TABLE") --- defines if all the files no matter what type of file it is will be in one table or it will be in separate tables (JOIN)
 	 * @ORM\DiscriminatorColumn(name="type", type="string") --- defines what column will specify what type file is
 	 * @ORM\DiscriminatorMap({"movie"="Movie", "pdf"="Pdf"}) --- bind (map) specific strings to child Classes (entities)




SUBSCRIBERS AND LISTENERS
 - Create a listener class (KernelResponseListener) then you have to create a function onYourWantedEvent (onKernelResponse)
		In that function do your logic 
 - Than you have to register this listener inside services.yaml via tags
		- App\Listeners\KernelResponseListener: 
			- tags:
				- {name: kernel.event_listener, event: kernel.response}
	- ---- This will setup your listener


CREATE OWN EVENT AND LISTENERS
 - Create two folders in src "Events" and "Listeners"
 - Inside these folder you can create event objects and listeners objects
	- --- We will create an VideoCreatedEvent and VideoCreatedListener classe
	- --- In services.yaml ---
	- App\Listeners\VideoCreatedListener:
		- tags: 
			- - {name: kernel.event_listener, event: video.created.event, method: onVideoCreatedEvent}
	- --- This will assign listener to event "video.created.event" and specify the method which should run inside the listener
	
	 - Creating the event class (VideoCreatedEvent) - this class has to use "Symfony\Contracts\EventDispatcher\Event;" and extend it
	 - Then create a public __construct($video) function into which u will pass your desired parameter - in this example its video object
	 	- --- inside __construct u pass the param -> $this->video = $video;

	 - Creating the listener class (VideoCreatedListener)
	 	- This class needs only the specified function in services.yaml or function with name like this => onYourEventName($event)
		- Pass to that function your event ($event) from which you can get your object (video) like this -> $event->video->param_of_the_video

	 - Executing the evet - Inside of the controller or after your desired logic (call to db etc.)
	 	- Create a variable $event -> $event = new VideoCreatedEvent($videl - your data);
		- Then you have to use in that controller --> use Symfony\Component\EventDispatcher\EventDispatcherInterface;
		- Assign dispatcher to variable and then call -> $this->dispatcher->dispatch($event, "video.created.event"); -- (first is $event object and second is name of event you want to execute);


SUBSCRIBERS
 - bin/console make:subscriber
 - Then choose which event subscriber should be subscribed to
 - Than you will have specific function for your event inside subscriber Class and there you can create your logic
 - Subscriber can listen to many events (you can specify them inside getSubscribedEvents)
	- Also you can handle one event with many functions -> "video.created.event" => [ ["name_of_function_1", PRIORITY_NUMBER (1)], ["name_of_function_2", 2] ]


AUTHENTICATION AND SECURITY DETAILS - REGISTER
 - bin/console make:user --- creates an user entity
 - bin/console make:form --- makes form for a user (register form)
	- use Symfony\Component\Validator\Constraints as Assert; --- use this inside a user entity
		- then you can specify details (rules) for specific fields ---> *@Assert\NotBlank() || *@Assert\Email() || *@Assert\Length(min=2, max=20) && add before class this -> *@UniqueEntity("email") (or your unique field) -> use Symfony\Bridge\Doctrine\Validator\Constraints\UniqueEntity;
	- Then edit generated Register form
	- THEN USE THIS CODE TO DISPLAY FORM AND HANDLE FORM
	- --- $passEncoder = use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;
		- $user = new SecurityUser();
		- $form = $this->createForm(RegisterUserType::class, $user);
		- $form->handleRequest($request);
		- if ($form->isSubmitted() && $form->isValid()){
			- $user->setPassword($passEncoder->encodePassword($user, $form->get("password")->getData()));
            	- $user->setEmail($form->get("email")->getData());
            $em->persist($user);
            $em->flush();
            $this->redirectToRoute("app_main");
        - }


AUTHENTICATION AND SECURITY DETAILS - LOGIN
 - security.yaml -> firewalls -> main
	- -> form_login:
		- login_path: login
		- check_path: login
		- username_parameter: 'email'
		- password_parameter: 'password'
		- csrf_token_generator: security.csrf.token_manager --> only if u want to use csrf (then u have to set enable_authenticator_manager=false)

LOGIN - REMEMBER ME
 - security.yaml -> firewalls -> main
	- -> remember_me:
			- secret: "%kernel.secret%"
			- lifetime: 604800 #1 week in seconds
			- #always_remember_me: true  --- only if u want always when user logs in to remember him
			- path: /


ACCESS COTROLL
@Security used on routed function of controller
 - TO USE THIS INSTALL -> composer require symfony/expression-language
 - With this u define if user has access to this page or not
 - use Sensio\Bundle\FrameworkExtraBundle\Configuration\Security;
 - @Security("user.getId() == video.getSecurityUser().getId()")
	- U can also specify user roles -> @Security("has_role('ROLE_ADMIN')")

security.yaml
 - access_control: and specify which path is accessable or not on specific ROLE

Functions in controller route functions
 - $this->denyAccessUnlessGranted('IS_AUTHENTICATED_FULLY') (IS_AUTHENTICATED_FULLY = is logged in --> you can use ROLE_ADMIN too)

TWIG
 - {% if is_granted('ROLE_ADMIN') %} .... {% endif %}


PHPUNIT TESTING SYMFONY
 - composer require --dev symfony/test-pack
 - You have two types of tests - Unit test and Functional tests
	- Unit tests are used to test specific functionality
	- Functional tests are used to test controllers
		- --- more at vids 96->106


EMAILING IN SYMFONY (symfony/mailer)
 - composer require symfony/mailer
	- Setup the mail server ins .env file
 - use Symfony\Component\Mailer\MailerInterface;
 - use Symfony\Component\Mime\Address as MailAddress;
 - use Symfony\Bridge\Twig\Mime\TemplatedEmail;
	- You have to create an a template for email
 - Then just use this
	- your_function(MailerInterface $mailer){
		- $email = (new TemplatedEmail())
			- ->from("your_email@from_where_mail_is.send")
			- ->to(new MailAddress("mail_to@send.mail", "Patrik Picka"))
			- ->subject("Order Confirmation")
			- ->htmlTemplate("emails/order-conf.html.twig")
			- ->context(["order" => ["name" => "Patrik Picka", "id" => 1, "email" => "opicaklp@gmail.com"]]);

		- $mailer->send($email);
	- }


SYMFONY FORM BUILDER
 - composer require symfony/form
 - bin/console make:form
	- --- This will create a formtype class 
 - Then in controller in which you want to display the form you have to set new Entity on which is the form bound to
  	- $video = new Video(); -> 
 	- $form = $this->createForm(VideoFormType::class, $video);
        - $form->handleRequest($request);
        - if ($form->isSubmitted() && $form->isValid()){
            - dump($form->getData());
            // return $this->redirectToRoute("app_main");
        - }
	- $form->createView(); ---> inside twig template just call {{form(your_variable_in_which_is_form_assigned)}}



RENDER AN HTML IN MULTIPLE TEMPLATES (RENDERING CONTROLLERS)
 - Create a controller/service and public function ---> inside that function make the logic u want and then return "$this->render("path_to_the_duplicating_content")" function
 - In twig templates in which u want to display the content use {{render(controller('App\\Controller(Services)\\YourController(YourService)::the_function_which_renders_the_content'))}}


TWIG
 - index of loop in for = loop.index (starts with 1) || loop.index0 (starts with 0)


CUSTOM ERROR PAGES
 - inside templates folder create bundles -> TwigBundle -> Exception -> error404.html.twig (error a číslo erroru).html.twig
 - after that you have to clear cache so execute this script inside cli -> bin/console cache:clear
 - --- Error pages will be shown only on production mode so if you want to see them in dev mode -> server_url/_error/error_code (404, 500 etc.)

CUSTOM TWIG EXTENSIONS ("Your string"|slugify)
 - bin/console make:twig-extension
 - getFilters ---> edit this to -> new TwigFilter("slugify", [$this (or namespace), "your_function_to_call")
 - then create your function and do your wanted logic


FLASH MESSAGES
 - in controller use "$this->addFlash("name", "message")
 - in twig {% for message in app.flashes("name") %}<p>{{message}}</p>{% endfor %}


 TRANSLATIONS 
 - composer require symfony/translation
	- There will be translations folder in root dir
	- And inside will be .yaml file with specific lang
 - TWIG -> {% trans %} your translatable text {% entrans %} || {{ 'translatable text'|trans }}
 - CONTROLLER -> use Symfony\Component\Translation\TranslatorInterface; -> $translator->trans('your translatable value')
 - ROUTES -> @Route({"en": "/login", "cs": "/prihlaseni"}, name="home") --- Then specify prefix in annotations.yaml

	- PLURALIZATION -> Texts inside messages.lang.yaml will be separated by "|" and starts with "{index_of_text}"
		- Then inside twig template -> {{'your.key.to.text'|transchoice(index_of_desired_text)}}
 - More about trans in vids 106->110
