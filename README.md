#MathSupportMatrix

The MathSupportMatrix (MSM) is a discovery and reference tool for pairing math content sources with accessible setups.

##About

The tool allows users to search for a combination of assistive technology, browser/readers, formats, and other attributes of their computing environment so as to discover how best to consume digital mathematics accessibly with said combination. 

We will seek to incorporate community feedback to identify more sources and setups as well as their respective benefits and issues. 

The site streamlines the process of adding new setups, versions of renderers, browsers/readers, platforms, and assistive technologies. 

Users will eventually contribute content sources that work well with one or more setups. Benetech staff, for now, will be responsible for approving submitted setups by testing them manually and then changing their status to an approved state that will allow these setups to be live on the website. User ratings and comments about setups could follow as well.

MSM is built with Ruby on Rails, CoffeeScript, and SASS.

##Setup

This command will clone the repository into a directory called math-support-matrix. This will be the directory we will be working in.

    git clone http://github.com/benetech/math-support-matrix

We use Vagrant. [Download and install Vagrant](http://www.vagrantup.com/downloads.html)

Vagrant uses VirtualBox.  [Download and install VirtualBox](https://www.virtualbox.org/wiki/Downloads)

You may need to make sure that VirtualBox's installation directory is in your path.


###Spin Up the Environment

Run the command ```vagrant up``` in the math-support-matrix directory. This will take a while to download the Linux distro, image it, install ruby/rails/other software. etc. Once it's done, you can login with ```vagrant ssh```. 

If you want to use other tools for ssh, just ssh to port 2222 on your local machine, and that's where ssh is listening to take you to the VirtualBox instance. The username and password are both ```vagrant ```.

Once you're in the VirtualBox, then issue the following commands.

    cd /vagrant
    rake db:drop db:create db:migrate db:seed
    rake db:drop db:create db:migrate db:seed RAILS_ENV=test

##Running

To run the server, do:

    vagrant ssh #if you have not already
    rails server

Once you do that, you can go to [http://localhost:3000/](http://localhost:3000/) and you should see the application's homepage.

Also, you may need to issue this in a separate terminal on your local box if the port forwarding for port 3000 on VirtualBox isn't setup:

    ssh -N -L 3000:localhost:3000 vagrant@localhost -p 2222

Again, the password is ```vagrant```.

To run the tests while you are developing, run ```guard``` in another terminal:

    guard


##Data model
To generate the data model diagram, you can do:

    railroady -o models.dot -M
    dot -Tpng models.dot > models.png

Here's a representation of it for [nomnoml](http://nomnoml.com)

    [<frame>Math Support Matrix data model|
      [User | id: int | email: string | admin: bool | timestamps]

      [Setup | id: int | title: str | notes: text | status_id: reference | file_format_id: reference | 
        platform_version_id: reference | browser_reader_version_id :  reference | assistive_technology_version_id : reference |
         renderer_version_id : reference  | affordance_id: reference | timestamps]
     
      [WorkflowStatus | id: int | title: str ]
      [WorkflowStatus] <- [Setup]

      [ContentSource | id: int | title: string | notes: text | timestamps]
      [ContentSourceSetup ] -> [Setup]
      [ContentSourceSetup ] -> [ContentSource]

      [FileFormat | id: int | title: string | notes: text | timestamps]
      [Setup] -> [FileFormat]

      [Platform | id: int | title: str | notes: text | timestamps]
      [PlatformVersion | id: int | version: float | notes: text | timestamps]
      [Platform] <- [PlatformVersion]
      [Setup] -> [PlatformVersion]


      [Renderer | id: int | title: str | notes: text | timestamps]
      [RendererVersion | id: int | version: float | notes: text | timestamps]
      [Renderer] <- [RendererVersion]
      [Setup] -> [RendererVersion]

      [BrowserReader | id: int | title: str | notes: text | timestamps]
      [BrowserReaderVersion | id: int | version: float | notes: text | timestamps]
      [BrowserReader] <- [BrowserReaderVersion]
      [Setup] -> [BrowserReaderVersion]

      [AssistiveTechnology | id: int | title: str | notes: text | timestamps]
      [AssistiveTechnologyVersion | id: int | title: str | notes: text | timestamps]
      [AssistiveTechnology] <- [AssistiveTechnologyVersion]
      [Setup] -> [AssistiveTechnologyVersion]

      [BrowserReaderRenderer] -> [Renderer]
      [BrowserReaderRenderer] -> [BrowserReader]

      [BrowserReaderFileFormat] -> [FileFormat]
      [BrowserReaderFileFormat] -> [BrowserReader]

      [PlatformBrowserReader] -> [Platform]
      [PlatformBrowserReader] -> [BrowserReader]

      [PlatformAssistiveTechnology] -> [AssistiveTechnology]
      [PlatformAssistiveTechnology] -> [Platform]

      [BrowserReaderAssistiveTechnology] -> [AssistiveTechnology]
      [BrowserReaderAssistiveTechnology] -> [BrowserReader]

      [Technology | id: int | title: str | notes: text |timestamps]
      [Feature | id: int | title: str | notes: text | timestamps]
      [Affordance | id: int | technology_id : reference | feature_id: reference | timestamps]
      [VerificationStatus | id: int | title: str ]
      [Capability | id: int | affordance_id:reference | setup_id: reference | verification_status_id: ref | timestamps]

      [Capability] -> [VerificationStatus]
      [Capability] -> [Setup]
      [Capability] -> [Affordance]
      [Affordance] -> [Feature]
      [Affordance] -> [Technology]

    ]

## Scaffolds

First, we initialized our generators and our user login system:

    rails g devise:install
    rails g devise user 
    rails g devise:views users
    #https://github.com/plataformatec/devise
    rake acts_as_taggable_on_engine:install:migrations
    #https://github.com/mbleigh/acts-as-taggable-on

Then we generated our scaffolds:

    #setup components
    rails g pizza_scaffold workflow_status title:string --force
    rails g pizza_scaffold file_format title:string notes:text  --force
    rails g pizza_scaffold platform title:string notes:text --force
    rails g pizza_scaffold platform_version platform:references version:float notes:text --force
    rails g pizza_scaffold renderer title:string notes:text --force
    rails g pizza_scaffold renderer_version renderer:references version:float  notes:text  --force
    rails g pizza_scaffold browser_reader title:string notes:text --force
    rails g pizza_scaffold browser_reader_version browser_reader:references version:float notes:text  --force
    rails g pizza_scaffold assistive_technology title:string notes:text --force
    rails g pizza_scaffold assistive_technology_version assistive_technology:references version:float notes:text  --force
    rails g pizza_scaffold setup platform_version:references \
        renderer_version:references browser_reader_version:references \
        assistive_technology_version:references \
        file_format:references workflow_status:references notes:text --force
    #content sources
    rails g pizza_scaffold content_source title:string notes:text --force
    rails g pizza_scaffold content_source_setup setup:references content_source:references --force
    #component suggestion
    rails g pizza_scaffold browser_reader_renderer browser_reader:references renderer:references --force
    rails g pizza_scaffold browser_reader_file_format browser_reader:references file_format:references --force
    rails g pizza_scaffold platform_browser_reader platform:references browser_reader:references --force
    rails g pizza_scaffold platform_assistive_technology platform:references assistive_technology:references --force 
    rails g pizza_scaffold browser_reader_assistive_technology browser_reader:references assistive_technology:references --force 
    #capability components
    rails g pizza_scaffold technology title:string notes:text --force
    rails g pizza_scaffold verification_status title:string --force
    rails g pizza_scaffold feature title:string notes:text --force
    rails g pizza_scaffold affordance feature:references technology:references --force
    rails g pizza_scaffold capability affordance:references setup:references verification_status:references --force

    #to generate only the controllers and views
    rails g pizza_controller workflow_status title:string --force
    rails g pizza_controller file_format title:string notes:text  --force
    rails g pizza_controller platform title:string notes:text --force
    rails g pizza_controller platform_version platform:references version:float notes:text --force
    rails g pizza_controller renderer title:string notes:text --force
    rails g pizza_controller renderer_version renderer:references version:float  notes:text  --force
    rails g pizza_controller browser_reader title:string notes:text --force
    rails g pizza_controller browser_reader_version browser_reader:references version:float notes:text  --force
    rails g pizza_controller assistive_technology title:string notes:text --force
    rails g pizza_controller assistive_technology_version assistive_technology:references version:float notes:text  --force
    rails g pizza_controller setup platform_version:references \
        renderer_version:references browser_reader_version:references \
        assistive_technology_version:references \
        file_format:references workflow_status:references notes:text --force
    #content sources
    rails g pizza_controller content_source title:string notes:text --force
    rails g pizza_controller content_source_setup setup:references content_source:references --force
    #component suggestion
    rails g pizza_controller browser_reader_renderer browser_reader:references renderer:references --force
    rails g pizza_controller browser_reader_file_format browser_reader:references file_format:references --force
    rails g pizza_controller platform_browser_reader platform:references browser_reader:references --force
    rails g pizza_controller platform_assistive_technology platform:references assistive_technology:references --force 
    rails g pizza_controller browser_reader_assistive_technology browser_reader:references assistive_technology:references --force 

    #capability components
    rails g pizza_controller technology title:string notes:text --force
    rails g pizza_controller verification_status title:string --force
    rails g pizza_controller feature title:string notes:text --force
    rails g pizza_controller affordance feature:references technology:references --force
    rails g pizza_controller capability affordance:references setup:references verification_status:references --force

##Components
- [RubyOnRails](http://rubyonrails.org/)
- [PostGRES](http://www.postgresql.org/)
- [rbenv](http://rbenv.org/) with [plugins](https://github.com/sstephenson/rbenv/wiki/Plugins) for gems, bundler, build, and binstubs
- [bundler](http://bundler.io/)
- [SASS](http://sass-lang.com/)
- [Coffeescript](http://coffeescript.org/)
- [Tenon](http://tenon.io)
- [Loggly](https://msmseeread.loggly.com/login/?next=/#)

##Links
- [benetech/math-support-matrix repo](http://github.com/benetech/math-support-matrix)
- [Benetech](http://benetech.org)
- [A11Y Guidelines](http://a11yproject.com/)
- [Tenon](http://tenon.io) for accessibility debugging and [Tenon Check](https://chrome.google.com/webstore/detail/tenon-check/bmibjbhkgepmnehjfhjaalkikngikhgj?hl=en-US) in Chrome
- [aXe](https://github.com/dequelabs/axe-core)
- [Sina's Links on Accessibility](http://www.sinabahram.com/resources.php)
- [ARIA in HTML](http://rawgit.com/w3c/aria-in-html/master/index.html) and [ARIA](http://www.w3.org/TR/wai-aria/states_and_properties#global_states)

##Contributors
- [@SinaBahram](https://twitter.com/SinaBahram)
- [@seereadnow](https://twitter.com/seereadnow)
