**Under heavy construction**

# Вступление

> Role models are important. <br/>
> -- Офицер Алекс Мёрфи / Робот-полицейский

Цель этого руководства заключается в представлении набора лучших практик и рецептов стиля оформления кода при разработке на Ruby on Rails 4. Оно дополняет уже существующее разрабатываемое сообществом руководство
[Руби: руководство по стилю оформления][ruby-style-guide].

Некоторые из приведенных здесь рекомендаций будут применимы только
для Rails 4.0+.

Вы можете создать копию этого руководства в форматах PDF или HTML при помощи
[Transmuter][].

Переводы данного руководства доступны на следующих языках:

* [английский (исходная версия)](https://github.com/bbatsov/rails-style-guide/blob/master/README.md)
* [китайский традиционный](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhTW.md)
* [китайский упрощенный](https://github.com/JuanitoFatas/rails-style-guide/blob/master/README-zhCN.md)
* [русский (данный документ)](https://github.com/arbox/rails-style-guide/blob/master/README-ruRU.md)
* [турецкий](https://github.com/tolgaavci/rails-style-guide/blob/master/README-trTR.md)
* [японский](https://github.com/satour/rails-style-guide/blob/master/README-jaJA.md)

# Руководство по стилю оформления Rails

Данное руководство по стилю оформления кода Rails рекомендует лучшие практики чтобы одни программисты Rails могли писать код который легко будет поддерживаться другими программистами. Руководство по стилю которое отражает реальное практики написания кода используется, а руководство, описывающее идеальные условия, но не отражающее реальность было отвергнуто сообществом, независимо от того насколько оно хорошо.

Руководство разделено на несколько секций связанных правил. Я пытался добавить обоснования правил(если обоснование пропущено, значит я посчитал что оно очевидно).

Я не пришел со всеми этими правилами из ниоткуда - они основаны на моей обширной карьереы в качестве профессионального разработчика, отзывах и предложениях от членов сообщества Rails и значительных ресурсов по программированию на Rails.

## Содержание

* [Конфигурация](#Конфигурация)
* [Маршрутизация](#Маршрутизация)
* [Контроллеры](#Контроллеры)
* [Модели](#Модели)
* [Миграции](#Миграции)
* [Представления](#Представления)
* [Интернационализация](#Интернационализация)
* [Assets](#assets)
* [Mailers](#mailers)
* [Bundler](#bundler)
* [Flawed Gems](#flawed-gems)
* [Управление процессами](#Управление-процессами)

## Конфигурация

* <a name="config-initializers"></a>
  Кладите код, выполняющийся при инициализации приложения в `config/initializers`.
  <sup>[[ссылка](#config-initializers)]</sup>

* <a name="gem-initializers"></a>
  Держите код инициализации каждого гема в отдельных файлах, названных так же как гем. Например `carrierwave.rb`, `active_admin.rb`, и т.д.
  <sup>[[ссылка](#gem-initializers)]</sup>

* <a name="dev-test-prod-configs"></a>

  Настройте среду окружения для разработки, тестов и деплоймента (в соответствующих файлах, лежащих в `config/environments/`)
  <sup>[[ссылка](#dev-test-prod-configs)]</sup>

        ```Ruby
        # config/environments/production.rb
        # Precompile additional assets (application.js, application.css, and all non-JS/CSS are already added)
        config.assets.precompile += %w( rails_admin/rails_admin.css rails_admin/rails_admin.js )
        ```

* <a name="app-config"></a>
  Держите конфигурацию общую для всех окружений в файле `config/application.rb`.
  <sup>[[ссылка](#app-config)]</sup>

* <a name="staging-like-prod"></a>
  Создайте дополнительную `staging` среду, которая очень близко напоминает среду`production`.
  <sup>[[ссылка](#staging-like-prod)]</sup>

## Маршрутизация

* <a name="member-collection-routes"></a>
  Если вам нужно добавить больше экшенов к RESTful ресурсу (они вам действительно необходимы?), используйте
  `member` и `collection` маршруты.
  <sup>[[ссылка](#member-collection-routes)]</sup>

    ```Ruby
    # плохо
    get 'subscriptions/:id/unsubscribe'
    resources :subscriptions

    # хорошо
    resources :subscriptions do
      get 'unsubscribe', on: :member
    end

    # плохо
    get 'photos/search'
    resources :photos

    # хорошо
    resources :photos do
      get 'search', on: :collection
    end
    ```

* <a name="many-member-collection-routes"></a>
	Если вам надо определить много `member/collection` маршрутов, 	 используйте альтернативный синтаксис блока.
  
  <sup>[[ссылка](#many-member-collection-routes)]</sup>

    ```Ruby
    resources :subscriptions do
      member do
        get 'unsubscribe'
        # more routes
      end
    end

    resources :photos do
      collection do
        get 'search'
        # more routes
      end
    end
    ```

* <a name="nested-routes"></a>
	Используйте вложенные маршруты чтобы лучше показать отношения между моделями.
  <sup>[[ссылка](#nested-routes)]</sup>

    ```Ruby
    class Post < ActiveRecord::Base
      has_many :comments
    end

    class Comments < ActiveRecord::Base
      belongs_to :post
    end

    # routes.rb
    resources :posts do
      resources :comments
    end
    ```

* <a name="namespaced-routes"></a>
  Используйте маршрутные пространства имен чтобы группировать связанные экшены.
  <sup>[[ссылка](#namespaced-routes)]</sup>

    ```Ruby
    namespace :admin do
      # Directs /admin/products/* to Admin::ProductsController
      # (app/controllers/admin/products_controller.rb)
      resources :products
    end
    ```

* <a name="no-wild-routes"></a>
  Никогда не используйте устаревший дикий маршрут контроллера. Этот маршрут сделает все экшены в каждом контроллере, доступными через запросы GET.
  <sup>[[ссылка](#no-wild-routes)]</sup>

    ```Ruby
    # очень плохо
    match ':controller(/:action(/:id(.:format)))'
    ```

* <a name="no-match-routes"></a>
  Не используйте `match` чтобы определить маршруты. Это удалено в Rails 4.
  <sup>[[ссылка](#no-match-routes)]</sup>

## Контроллеры

* <a name="skinny-controllers"></a>
Держите контроллеры "тощими" - они должны только получать данные из видов и не должны содержать какую-либо бизнес-логику (вся бизнес-логика должна быть в моделях.
  <sup>[[ссылка](#skinny-controllers)]</sup>

* <a name="one-method"></a>
  Каждый экшен в контроллере должен (в идеале) вызывать только один метод, кроме initial find or new.
  <sup>[[ссылка](#one-method)]</sup>

* <a name="shared-instance-variables"></a>
  Обменивайтесь не более двумя переменными экземпляра между контроллером и видом.
  <sup>[[ссылка](#shared-instance-variables)]</sup>

## Модели

* <a name="model-classes"></a>
  Introduce non-ActiveRecord model classes freely.
  <sup>[[ссылка](#model-classes)]</sup>

* <a name="meaningful-model-names"></a>
	Называйте модели понятными (но короткими) именами без аббревиатур.
  <sup>[[ссылка](#meaningful-model-names)]</sup>

* <a name="activeattr-gem"></a>
	Если вам нужны объекты модели которые поддерживают поведение ActiveRecord (например, валидацию), используйте [ActiveAttr](https://github.com/cgriego/active_attr) гем.
  <sup>[[ссылка](#activeattr-gem)]</sup>

    ```Ruby
    class Message
      include ActiveAttr::Model

      attribute :name
      attribute :email
      attribute :content
      attribute :priority

      attr_accessible :name, :email, :content

      validates :name, presence: true
      validates :email, format: { with: /\A[-a-z0-9_+\.]+\@([-a-z0-9]+\.)+[a-z0-9]{2,4}\z/i }
      validates :content, length: { maximum: 500 }
    end
    ```

    Более подробный пример смотрите в скринкасте
    [RailsCast on the subject](http://railscasts.com/episodes/326-activeattr).

### ActiveRecord

* <a name="keep-ar-defaults"></a>
	Избегайте изменять умолчания ActiveRecord (имена таблиц, первичные ключи и т.д.), если только у вас нет веских причин делать это (например если база данных управляется не вами)
  <sup>[[ссылка](#keep-ar-defaults)]</sup>

    ```Ruby
    # плохо (не делайте так, если вы можете изменить схему)
    class Transaction < ActiveRecord::Base
      self.table_name = 'order'
      ...
    end
    ```

* <a name="macro-style-methods"></a>
	Группируйте макро методы (`has_many`, `validates`, и т.д.) в начале определения класса
  <sup>[[ссылка](#macro-style-methods)]</sup>

    ```Ruby
    class User < ActiveRecord::Base
      # скоуп по умолчанию должен быть первым (если он есть)
      default_scope { where(active: true) }

      # далее идут константы
      GENDERS = %w(male female)

      # после них кладем макросы работающие с атрибутами
      attr_accessor :formatted_date_of_birth

      attr_accessible :login, :first_name, :last_name, :email, :password

      # затем пишем ассоциации
      belongs_to :country

      has_many :authentications, dependent: :destroy

      # и валидации
      validates :email, presence: true
      validates :username, presence: true
      validates :username, uniqueness: { case_sensitive: false }
      validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
      validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true}

      # далее идут коллбеки
      before_save :cook
      before_save :update_username_lower

      # остальные макросы (например devise должны быть после всех коллбеков
      ...
    end
    ```

* <a name="has-many-through"></a>
  Предпочитайте ассоциацию `has_many :through` ассоциацям вида `has_and_belongs_to_many`. Использование  `has_many :through` позволяет иметь дополнительные атрибуты и валидаци при объединении моделей.
  <sup>[[ссылка](#has-many-through)]</sup>

    ```Ruby
    # не очень хорошо (использование has_and_belongs_to_many)
    class User < ActiveRecord::Base
      has_and_belongs_to_many :groups
    end

    class Group < ActiveRecord::Base
      has_and_belongs_to_many :users
    end

    # предпочтительно - использование has_many :through
    class User < ActiveRecord::Base
      has_many :memberships
      has_many :groups, through: :memberships
    end

    class Membership < ActiveRecord::Base
      belongs_to :user
      belongs_to :group
    end

    class Group < ActiveRecord::Base
      has_many :memberships
      has_many :users, through: :memberships
    end
    ```

* <a name="read-attribute"></a>
  Лучше использовать `self[:attribute]` чем `read_attribute(:attribute)`.
  <sup>[[ссылка](#read-attribute)]</sup>

    ```Ruby
    # плохо
    def amount
      read_attribute(:amount) * 100
    end

    # хорошо
    def amount
      self[:amount] * 100
    end
    ```

* <a name="write-attribute"></a>
  Всегда используйте новые ["сексуальные" валидации](http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/).
  <sup>[[ссылка](#write-attribute)]</sup>

    ```Ruby
    # плохо
    validates_presence_of :email

    # хорошо
    validates :email, presence: true
    ```

* <a name="sexy-validations"></a>
	Когда пользовательская валидация используется чаще чем один раз или является неким регулярным выражением, создайте файл пользовательской валидации.
  <sup>[[ссылка](#sexy-validations)]</sup>

    ```Ruby
    # плохо
    class Person
      validates :email, format: { with: /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i }
    end

    # хорошо
    class EmailValidator < ActiveModel::EachValidator
      def validate_each(record, attribute, value)
        record.errors[attribute] << (options[:message] || 'is not a valid email') unless value =~ /\A([^@\s]+)@((?:[-a-z0-9]+\.)+[a-z]{2,})\z/i
      end
    end

    class Person
      validates :email, email: true
    end
    ```

* <a name="custom-validator-file"></a>
  Держите свои валидаторы в папке `app/validators`.
  <sup>[[ссылка](#custom-validator-file)]</sup>

* <a name="app-validators"></a>
  Подумайте о создании гема из пользовательских валидаторов, если вы поддерживаете несколько похожих приложений или ваши валидаторы имеют достаточно общий характер.
  <sup>[[ссылка](#app-validators)]</sup>

* <a name="custom-validators-gem"></a>
  Свободно используйте пространства имен (скоупы).
  <sup>[[ссылка](#custom-validators-gem)]</sup>

    ```Ruby
    class User < ActiveRecord::Base
      scope :active, -> { where(active: true) }
      scope :inactive, -> { where(active: false) }

      scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
    end
    ```

* <a name="named-scopes"></a>
	Оборачивайте именованные скоупы в `лямбды` чтобы инициализировать их "лениво". (это только рекомендация в Rails 3, но является обязательным в Rails 4)
  <sup>[[ссылка](#named-scopes)]</sup>

    ```Ruby
    # плохо
    class User < ActiveRecord::Base
      scope :active, where(active: true)
      scope :inactive, where(active: false)

      scope :with_orders, joins(:orders).select('distinct(users.id)')
    end

    # good
    class User < ActiveRecord::Base
      scope :active, -> { where(active: true) }
      scope :inactive, -> { where(active: false) }

      scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
    end
    ```

* <a name="named-scope-class"></a>
	Когда именованный скоуп определен и через лямбда-функцию и параметры становятся слишком сложными, лучше создать метод класса который обслуживает этот скоуп и будет возвращать объект `ActiveRecord::Relation`. Возможно вы сможете определить более простой скоуп.
  <sup>[[ссылка](#named-scope-class)]</sup>

    ```Ruby
    class User < ActiveRecord::Base
      def self.with_orders
        joins(:orders).select('distinct(users.id)')
      end
    end
    ```

* <a name="beware-update-attribute"></a>
Остерегайтесь поведения метода [`update_attribute`](http://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-update_attribute). Он не запускает валидацию модели (в отличие от `update_attributes`) и легко может нарушить состояние модели.
  <sup>[[ссылка](#beware-update-attribute)]</sup>

* <a name="user-friendly-urls"></a>
	Используйте понятные человеку адреса (ЧПУ). Покажите описателный атрибут модели, а не id. Есть много способов сделать это.
  <sup>[[ссылка](#user-friendly-urls)]</sup>

  * Переопределите метод модели `to_param`. Этот метод используется Rails для построения url объекта. По умолчанию этот метод возвращает `id` объекта в виде String. Он может быть переопределен, чтобы показать более читаемый атрибут.

        ```Ruby
        class Person
          def to_param
            "#{id} #{name}".parameterize
          end
        end
        ```

    Для того чтобы преобразовать это в читаемый вид, `parametereize` должен быть применен к строке. `id` объекта должен идти вначале для того чтобы запись могла быть найдена с помощью метода ActiveRecord `find`
    
  * Используйте `friendly_id` гем. Он позволяет создавать ЧПУ используя описательный атрибут модели, а не `id`.

        ```Ruby
        class Person
          extend FriendlyId
          friendly_id :name, use: :slugged
        end
        ```

  Посмотрите [документацию гема](https://github.com/norman/friendly_id)
  для более подробной информации.

* <a name="find-each"></a>
  Используйте `find_each` для перебора коллекции объектов AR. Цикл по коллекции записей из базы данных (используя `all` метод, например) является очень неэффективным, так как он будет пытаться загрузить все объекты сразу. В данном случае пакетная обработка позволяет работать с записями в партиях, что значительно снижает потребление памяти.
  <sup>[[ссылка](#find-each)]</sup>


    ```Ruby
    # плохо
    Person.all.each do |person|
      person.do_awesome_stuff
    end

    Person.where("age > 21").each do |person|
      person.party_all_night!
    end

    # хорошо
    Person.all.find_each do |person|
      person.do_awesome_stuff
    end

    Person.where("age > 21").find_each do |person|
      person.party_all_night!
    end
    ```

* <a name="before_destroy"></a>
  Так как [Rails создает коллбеки для зависимых ассоциаций](https://github.com/rails/rails/issues/3458), всегда вызывайте  `before_destroy` коллбеки которые выполняют проверку с `prepend: true`.

  ```Ruby
  # плохо (роли будут удалены автоматически, даже если super_admin? - true)
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end

  # хорошо
  has_many :roles, dependent: :destroy

  before_destroy :ensure_deletable, prepend: true

  def ensure_deletable
    fail "Cannot delete super admin." if super_admin?
  end
  ```

## Миграции


* <a name="schema-version"></a>
  Держите файл `schema.rb` (или `structure.sql`) под управлением системы контроля версий.
<sup>[[ссылка](#schema-version)]</sup>

* <a name="db-schema-load"></a>
  Используйте `rake db:schema:load` , а не `rake db:migrate` чтобы инициализировать пустую базу.
<sup>[[ссылка](#db-schema-load)]</sup>

* <a name="default-migration-values"></a>
  Выносите значения по умолчанию на уровень базы данных, а не на уровень приложения.
<sup>[[ссылка](#default-migration-values)]</sup>

  ```Ruby
  # плохо - значения по умолчанию вынесены в приложение.
  def amount
    self[:amount] or 0
  end
  ```

  Несмотря на то что управление значениями по умолчанию предлагается многими Rails разработчиками, это очень опасный подход, который оставляет данные уязвимыми для ошибок приложений. Вы должны учитывать тот факт что большинство нетривиальных приложений делять базы данных с другими приложениями, и управлять целостностью данных в базе не могут.

* <a name="foreign-key-constraints"></a>
 Принудительное ограничения внешнего ключа. Хотя ActiveRecord не поддерживает их в естественном виде, есть некоторые прекрасные сторонние гемы, как [schema_plus](https://github.com/lomba/schema_plus) и  [foreigner](https://github.com/matthuhiggins/foreigner).
<sup>[[ссылка](#foreign-key-constraints)]</sup>

* <a name="change-vs-up-down"></a>
  Когда пишете конструктивные миграции (добавляющие столбцы или таблицы),
используйте `change` метод, а не `up` and `down`.
<sup>[[ссылка](#change-vs-up-down)]</sup>

  ```Ruby
  # устаревший подход
  class AddNameToPeople < ActiveRecord::Migration
    def up
      add_column :people, :name, :string
    end

    def down
      remove_column :people, :name
    end
  end

  # предпочтительный подход.
  class AddNameToPeople < ActiveRecord::Migration
    def change
      add_column :people, :name, :string
    end
  end
  ```

* <a name="no-model-class-migrations"></a>
  Не используйте классы модели в миграции. Классы модели постоянно развиваются и в какой-то момент в будущем миграции которые работали, могут перестать работать, из-за изменений в используемых моделях.
<sup>[[ссылка](#no-model-class-migrations)]</sup>

## Views

* <a name="no-direct-model-view"></a>
  Never call the model layer directly from a view.
<sup>[[ссылка](#no-direct-model-view)]</sup>

* <a name="no-complex-view-formatting"></a>
  Never make complex formatting in the views, export the formatting to a method
  in the view helper or the model.
<sup>[[ссылка](#no-complex-view-formatting)]</sup>

* <a name="partials"></a>
  Mitigate code duplication by using partial templates and layouts.
<sup>[[ссылка](#partials)]</sup>

## Internationalization

* <a name="locale-texts"></a>
  No strings or other locale specific settings should be used in the views,
  models and controllers. These texts should be moved to the locale files in the
  `config/locales` directory.
<sup>[[ссылка](#locale-texts)]</sup>

* <a name="translated-labels"></a>
  When the labels of an ActiveRecord model need to be translated, use the
  `activerecord` scope:
<sup>[[ссылка](#translated-labels)]</sup>

  ```
  en:
    activerecord:
      models:
        user: Member
      attributes:
        user:
          name: 'Full name'
  ```

  Then `User.model_name.human` will return "Member" and
  `User.human_attribute_name("name")` will return "Full name". These
  translations of the attributes will be used as labels in the views.


* <a name="organize-locale-files"></a>
  Separate the texts used in the views from translations of ActiveRecord
  attributes. Place the locale files for the models in a folder `models` and the
  texts used in the views in folder `views`.
<sup>[[ссылка](#organize-locale-files)]</sup>

  * When organization of the locale files is done with additional directories,
    these directories must be described in the `application.rb` file in order
    to be loaded.

      ```Ruby
      # config/application.rb
      config.i18n.load_path += Dir[Rails.root.join('config', 'locales', '**', '*.{rb,yml}')]
      ```

* <a name="shared-localization"></a>
  Place the shared localization options, such as date or currency formats, in
  files under the root of the `locales` directory.
<sup>[[ссылка](#shared-localization)]</sup>

* <a name="short-i18n"></a>
  Use the short form of the I18n methods: `I18n.t` instead of `I18n.translate`
  and `I18n.l` instead of `I18n.localize`.
<sup>[[ссылка](#short-i18n)]</sup>

* <a name="lazy-lookup"></a>
  Use "lazy" lookup for the texts used in views. Let's say we have the following
  structure:
<sup>[[ссылка](#lazy-lookup)]</sup>

  ```
  en:
    users:
      show:
        title: 'User details page'
  ```

  The value for `users.show.title` can be looked up in the template
  `app/views/users/show.html.haml` like this:

  ```Ruby
  = t '.title'
  ```

* <a name="dot-separated-keys"></a>
  Use the dot-separated keys in the controllers and models instead of specifying
  the `:scope` option. The dot-separated call is easier to read and trace the
  hierarchy.
<sup>[[ссылка](#dot-separated-keys)]</sup>

  ```Ruby
  # use this call
  I18n.t 'activerecord.errors.messages.record_invalid'

  # instead of this
  I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]
  ```

* <a name="i18n-guides"></a>
  More detailed information about the Rails i18n can be found in the [Rails
  Guides](http://guides.rubyonrails.org/i18n.html)
<sup>[[ссылка](#i18n-guides)]</sup>

## Assets

Use the [assets pipeline](http://guides.rubyonrails.org/asset_pipeline.html) to leverage organization within
your application.

* <a name="reserve-app-assets"></a>
  Reserve `app/assets` for custom stylesheets, javascripts, or images.
<sup>[[ссылка](#reserve-app-assets)]</sup>

* <a name="lib-assets"></a>
  Use `lib/assets` for your own libraries that don’t really fit into the
  scope of the application.
<sup>[[ссылка](#lib-assets)]</sup>

* <a name="vendor-assets"></a>
  Third party code such as [jQuery](http://jquery.com/) or
  [bootstrap](http://twitter.github.com/bootstrap/) should be placed in
  `vendor/assets`.
<sup>[[ссылка](#vendor-assets)]</sup>

* <a name="gem-assets"></a>
  When possible, use gemified versions of assets (e.g.
  [jquery-rails](https://github.com/rails/jquery-rails),
  [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails),
  [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass),
  [zurb-foundation](https://github.com/zurb/foundation)).
<sup>[[ссылка](#gem-assets)]</sup>

## Mailers

* <a name="mailer-name"></a>
  Name the mailers `SomethingMailer`. Without the Mailer suffix it isn't
  immediately apparent what's a mailer and which views are related to the
  mailer.
<sup>[[ссылка](#mailer-name)]</sup>

* <a name="html-plain-email"></a>
  Provide both HTML and plain-text view templates.
<sup>[[ссылка](#html-plain-email)]</sup>

* <a name="enable-delivery-errors"></a>
  Enable errors raised on failed mail delivery in your development environment.
  The errors are disabled by default.
<sup>[[ссылка](#enable-delivery-errors)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.raise_delivery_errors = true
  ```

* <a name="local-smtp"></a>
  Use a local SMTP server like
  [Mailcatcher](https://github.com/sj26/mailcatcher) in the development
  environment.
<sup>[[ссылка](#local-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.smtp_settings = {
    address: 'localhost',
    port: 1025,
    # more settings
  }
  ```

* <a name="default-hostname"></a>
  Provide default settings for the host name.
<sup>[[ссылка](#default-hostname)]</sup>

  ```Ruby
  # config/environments/development.rb
  config.action_mailer.default_url_options = { host: "#{local_ip}:3000" }

  # config/environments/production.rb
  config.action_mailer.default_url_options = { host: 'your_site.com' }

  # in your mailer class
  default_url_options[:host] = 'your_site.com'
  ```

* <a name="url-not-path-in-email"></a>
  If you need to use a link to your site in an email, always use the `_url`, not
  `_path` methods. The `_url` methods include the host name and the `_path`
  methods don't.
<sup>[[ссылка](#url-not-path-in-email)]</sup>

  ```Ruby
  # плохо
  You can always find more info about this course
  = link_to 'here', course_path(@course)

  # good
  You can always find more info about this course
  = link_to 'here', course_url(@course)
  ```

* <a name="email-addresses"></a>
  Format the from and to addresses properly. Use the following format:
<sup>[[ссылка](#email-addresses)]</sup>

  ```Ruby
  # in your mailer class
  default from: 'Your Name <info@your_site.com>'
  ```

* <a name="delivery-method-test"></a>
  Make sure that the e-mail delivery method for your test environment is set to
  `test`:
<sup>[[ссылка](#delivery-method-test)]</sup>

  ```Ruby
  # config/environments/test.rb

  config.action_mailer.delivery_method = :test
  ```

* <a name="delivery-method-smtp"></a>
  The delivery method for development and production should be `smtp`:
<sup>[[ссылка](#delivery-method-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb, config/environments/production.rb

  config.action_mailer.delivery_method = :smtp
  ```

* <a name="inline-email-styles"></a>
  When sending html emails all styles should be inline, as some mail clients
  have problems with external styles. This however makes them harder to maintain
  and leads to code duplication. There are two similar gems that transform the
  styles and put them in the corresponding html tags:
  [premailer-rails](https://github.com/fphilipe/premailer-rails) and
  [roadie](https://github.com/Mange/roadie).
<sup>[[ссылка](#inline-email-styles)]</sup>

* <a name="background-email"></a>
  Sending emails while generating page response should be avoided. It causes
  delays in loading of the page and request can timeout if multiple email are
  sent. To overcome this emails can be sent in background process with the help
  of [sidekiq](https://github.com/mperham/sidekiq) gem.
<sup>[[ссылка](#background-email)]</sup>

## Bundler

* <a name="dev-test-gems"></a>
  Put gems used only for development or testing in the appropriate group in the
  Gemfile.
<sup>[[ссылка](#dev-test-gems)]</sup>

* <a name="only-good-gems"></a>
  Use only established gems in your projects. If you're contemplating on
  including some little-known gem you should do a careful review of its source
  code first.
<sup>[[ссылка](#only-good-gems)]</sup>

* <a name="os-specific-gemfile-locks"></a>
  OS-specific gems will by default result in a constantly changing
  `Gemfile.lock` for projects with multiple developers using different operating
  systems.  Add all OS X specific gems to a `darwin` group in the Gemfile, and
  all Linux specific gems to a `linux` group:
<sup>[[ссылка](#os-specific-gemfile-locks)]</sup>

  ```Ruby
  # Gemfile
  group :darwin do
    gem 'rb-fsevent'
    gem 'growl'
  end

  group :linux do
    gem 'rb-inotify'
  end
  ```

  To require the appropriate gems in the right environment, add the
  following to `config/application.rb`:

  ```Ruby
  platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
  Bundler.require(platform)
  ```

* <a name="gemfile-lock"></a>
  Do not remove the `Gemfile.lock` from version control. This is not some
  randomly generated file - it makes sure that all of your team members get the
  same gem versions when they do a `bundle install`.
<sup>[[ссылка](#gemfile-lock)]</sup>

## Flawed Gems

This is a list of gems that are either problematic or superseded by
other gems. You should avoid using them in your projects.

* [rmagick](http://rmagick.rubyforge.org/) - this gem is notorious for its memory consumption. Use  [minimagick](https://github.com/probablycorey/mini_magick) instead.


*  [autotest](http://www.zenspider.com/ZSS/Products/ZenTest/) - old solution for running tests automatically. Far  inferior to [guard](https://github.com/guard/guard) and [watchr](https://github.com/mynyml/watchr).


*  [rcov](https://github.com/relevance/rcov) - code coverage tool, not  compatible with Ruby 1.9. Use
  [SimpleCov](https://github.com/colszowka/simplecov) instead.


*  [therubyracer](https://github.com/cowboyd/therubyracer) - the use of  this gem in production is strongly discouraged as it uses a very large amount of
  memory. I'd suggest using `node.js` instead.


This list is also a work in progress. Please, let me know if you know
other popular, but flawed gems.

## Управление процессами

*  <a name="foreman"></a>
   Если в вашем проекте есть много зависимостей от внешних процессов, применяйте
   библиотеку [foreman](https://github.com/ddollar/foreman) для управления ими.
   <sup>[[ссылка](#foreman)]</sup>

# Дополнительные источники

There are a few excellent resources on Rails style, that you should
consider if you have time to spare:

*  [The Rails 4 Way](http://www.amazon.com/The-Rails-Addison-Wesley-Professional-Ruby/dp/0321944275)* [Ruby on Rails Guides](http://guides.rubyonrails.org/)
* [The RSpec Book](http://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](http://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)


# Contributing

Ничто, описанное в этом руководстве, не высечено в камне. И я очень хотел бы
сотрудничать со всеми, кто интересуется стилистикой оформления кода Rails,
чтобы мы смогли вместе создать ресурс, который был бы полезен для всего сообщества
программистов на Руби.

Не стесняйтесь создавать отчеты об ошибках и присылать мне запросы на интеграцию
вашего кода. И заранее большое спасибо за вашу помощь!

Вы можете поддержать проект (и РубоКоп) денежным взносом
при помощи [gittip](https://www.gittip.com/bbatsov).

[![Дай Gittip](https://rawgithub.com/twolfson/gittip-badge/0.2.0/dist/gittip.png)](https://www.gittip.com/bbatsov)


## Как сотрудничать в проекте?

Это просто! Просто следуйте [руководству по сотрудничеству
](https://github.com/bbatsov/rails-style-guide/blob/master/CONTRIBUTING.md).

# Лицензирование

![Creative Commons License](http://i.creativecommons.org/l/by/3.0/88x31.png)
Данная работа опубликована на условиях лицензии [Creative Commons Attribution 3.0
Unported License](http://creativecommons.org/licenses/by/3.0/deed.en_US)

# Расскажи другому

Создаваемое сообществом руководство по стилю оформления будет малопригодным для
сообщества, которое об этом руководстве ничего не знает. Делитесь ссылками на
это руководство с вашими друзьями и коллегами доступными вам средствами. Каждый
получаемый нами комментарий, предложение или мнение сделает это руководство еще
чуточку лучше. А ведь мы хотим самое лучшее руководство из возможных, не так ли?

Всего,<br/>
[Божидар](https://twitter.com/bbatsov)

<!--- Links -->
[ruby-style-guide]: https://github.com/arbox/ruby-style-guide/blob/master/README-ruRU.md
[transmutter]: https://github.com/TechnoGate/transmuter
