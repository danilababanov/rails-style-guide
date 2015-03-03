# Вступление

> Ролевые модели важны. <br/>
> -- Офицер Алекс Мёрфи / Робот-полицейский

Целью этого руководства является распространение набора проверенных практик
и стилистических рекомендаций при разработки приложений с помощью Ruby on Rails
(для версий 3 и 4). Это руководство дополняет уже существующий сборник:
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

Настоящее руководство по стилю рекомендует лучшие практики оформленя, благодаря
которым обычные разработчики на Rails смогут писать код, который с легкостью
будут поддерживать другие обычные программисты. Руководство по оформлению,
которое отражает повседневные реалии, будет применяться постоянно,
а руководство, стремящееся к идеалу, который не принимается обычными людьми,
подвергается риску вообще быть забытым. При этом абсолютно не важно, насколько
хорошим оно является.

Данное руководство разделено на несколько частей, состоящий из связанных
по смыслу правил. В каждом случае я попытался обосновать появление этих правил
(объяснение опущено в ситуациях, когда я посчитал его очевидным).

Все эти правила не появились из пустоты, они по большей части основываются
на моем собственном обширном профессиональном опыте в качестве разработчика ПО,
отзывах и предложениях других членов сообщества разработчиков на Rails
и различных общепризнанных источниках по разработке Rails-приложений.

## Содержание

* [Конфигурация](#Конфигурация)
* [Маршрутизация](#Маршрутизация)
* [Контроллеры](#Контроллеры)
* [Модели](#Модели)
  * [ActiveRecord](#Activerecord)
  * [Запросы ActiveRecord](#Запросы-activerecord)
* [Миграции](#Миграции)
* [Представления](#Представления)
* [Интернационализация](#Интернационализация)
* [Ресурсы](#Ресурсы)
* [Почтовые модули](#Почтовые-модули)
* [Время](#Время)
* [Bundler](#Bundler)
* [Гемы с дефектами](#Гемы-с-дефектами)
* [Управление процессами](#Управление-процессами)

## Конфигурация

* <a name="config-initializers"></a>
  Код для инициализации приложения помещайте в директорию `config/initializers/`.
  Код в этой директории выполняется при запуске приложения.
  <sup>[[ссылка](#config-initializers)]</sup>

* <a name="gem-initializers"></a>
  Для каждого гема записывайте код инициализации в одноименный отдельный файл.
  Например, `carrierwave.rb`, `active_admin.rb` и т.д.
  <sup>[[ссылка](#gem-initializers)]</sup>

* <a name="dev-test-prod-configs"></a>
  Уточните настройки для рабочего, тестового и промышленного окружений
  в соответствующих файлах в директории `config/environments/`.
  <sup>[[ссылка](#dev-test-prod-configs)]</sup>

  * Mark additional assets for precompilation (if any):

    ```Ruby
    # config/environments/production.rb
    # Precompile additional assets (application.js, application.css,
    # and all non-JS/CSS are already added)
    config.assets.precompile += %w( rails_admin/rails_admin.css rails_admin/rails_admin.js )
    ```

* <a name="app-config"></a>
  Сохраняйте настройки, которые относятся ко всем окружениям, в файле
  `config/application.rb`.
  <sup>[[ссылка](#app-config)]</sup>

* <a name="staging-like-prod"></a>
  Создайте дополнительное окружение `staging`, которое будет очень схоже с
  вашим окружением `production`.
  <sup>[[ссылка](#staging-like-prod)]</sup>

## Маршрутизация

* <a name="member-collection-routes"></a>
  Если вам требуется добавить дополнительные действия к ресурсу REST (и вы
  уверены, что это вам абсолютно нужно), то используйте пути `member`
  и `collection`.
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
  If you need to define multiple `member/collection` routes use the alternative
  block syntax.
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
  Use nested routes to express better the relationship between ActiveRecord
  models.
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
  Use namespaced routes to group related actions.
  <sup>[[ссылка](#namespaced-routes)]</sup>

    ```Ruby
    namespace :admin do
      # Directs /admin/products/* to Admin::ProductsController
      # (app/controllers/admin/products_controller.rb)
      resources :products
    end
    ```

* <a name="no-wild-routes"></a>
  Never use the legacy wild controller route. This route will make all actions
  in every controller accessible via GET requests.
  <sup>[[ссылка](#no-wild-routes)]</sup>

    ```Ruby
    # очень плохо
    match ':controller(/:action(/:id(.:format)))'
    ```

* <a name="no-match-routes"></a>
  Don't use `match` to define any routes. It's removed from Rails 4.
  <sup>[[ссылка](#no-match-routes)]</sup>

## Контроллеры

* <a name="skinny-controllers"></a>
  Поддерживайте код контроллеров обозримым, контроллеры должны лишь получать
  данные для шаблонов и не должны реализовывать бизнес-логику. Вся бизнес-логика
  вашего приложения должна по определению реализовываться в моделях.
  <sup>[[ссылка](#skinny-controllers)]</sup>

* <a name="one-method"></a>
  Каждое действие в котроллере должно (в идеале) вызывать не более одного
  другого метода (кроме `find` или `new`).
  <sup>[[ссылка](#one-method)]</sup>

* <a name="shared-instance-variables"></a>
  Старайтесь не передавать более двух переменных из контроллера в шаблон.
  <sup>[[ссылка](#shared-instance-variables)]</sup>

## Модели

* <a name="model-classes"></a>
  Без зазрений совести используйте модели, не базирующиеся на ActiveRecord.
  <sup>[[ссылка](#model-classes)]</sup>

* <a name="meaningful-model-names"></a>
  Называйте модели говорящими (но короткими) именами без сокращений.
  <sup>[[ссылка](#meaningful-model-names)]</sup>

* <a name="activeattr-gem"></a>
  Если вам нужна модель, поддерживающая некоторые аспекты ActiveRecord
  (например, валидации), без привязки к работе с БД используйте библиотеку
  [ActiveAttr](https://github.com/cgriego/active_attr)
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

    Более подробный пример (на английском языке) вы найдете здесь:
    [RailsCast on the subject](http://railscasts.com/episodes/326-activeattr).

### ActiveRecord

* <a name="keep-ar-defaults"></a>
  Не меняйте стандартных значений ActiveRecord (например, наименования таблиц,
  первичных ключей и т.д.) без особой нужды. Это оправдано в таких случаях,
  когда вы работаете с базой данных, схему которой нет возможности изменить.
  <sup>[[ссылка](#keep-ar-defaults)]</sup>

    ```Ruby
    # плохо (не делайте так, если вы можете изменить схему)
    class Transaction < ActiveRecord::Base
      self.table_name = 'order'
      ...
    end
    ```

* <a name="macro-style-methods"></a>
  Группируйте макро-методы (`has_many`, `validates` и т.д.) в начале определения
  класса.
  <sup>[[ссылка](#macro-style-methods)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    # keep the default scope first (if any)
    default_scope { where(active: true) }

    # constants come up next
    COLORS = %w(red green blue)

    # afterwards we put attr related macros
    attr_accessor :formatted_date_of_birth

    attr_accessible :login, :first_name, :last_name, :email, :password

    # followed by association macros
    belongs_to :country

    has_many :authentications, dependent: :destroy

    # and validation macros
    validates :email, presence: true
    validates :username, presence: true
    validates :username, uniqueness: { case_sensitive: false }
    validates :username, format: { with: /\A[A-Za-z][A-Za-z0-9._-]{2,19}\z/ }
    validates :password, format: { with: /\A\S{8,128}\z/, allow_nil: true}

    # next we have callbacks
    before_save :cook
    before_save :update_username_lower

    # other macros (like devise's) should be placed after the callbacks

    ...
  end
  ```

* <a name="has-many-through"></a>
  Используйте преимущественно `has_many :through` вместо
  `has_and_belongs_to_many`. Применение ассоциации `has_many :through`
  дает вам большую свободу в определении дополнительных атрибутов и задании
  валидаций на модели объединения.
  <sup>[[ссылка](#has-many-through)]</sup>

    ```Ruby
    # не особо (применяется has_and_belongs_to_many)
    class User < ActiveRecord::Base
      has_and_belongs_to_many :groups
    end

    class Group < ActiveRecord::Base
      has_and_belongs_to_many :users
    end

    # предпочтительное решение (применяется has_many :through)
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
  Используйте `self[:attribute]` вместо `read_attribute(:attribute)`.
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
  Prefer `self[:attribute] = value` over `write_attribute(:attribute, value)`.
  <sup>[[ссылка](#write-attribute)]</sup>

  ```Ruby
  # плохо
  def amount
    write_attribute(:amount, 100)
  end

  # хорошо
  def amount
    self[:amount] = 100
  end
  ```

* <a name="sexy-validations"></a>
  Always use the new ["sexy" validations
  ](http://thelucid.com/2010/01/08/sexy-validation-in-edge-rails-rails-3/).
  <sup>[[ссылка](#sexy-validations)]</sup>

  ```Ruby
  # плохо
  validates_presence_of :email
  validates_length_of :email, maximum: 100

  # хорошо
  validates :email, presence: true, length: { maximum: 100 }
  ```

* <a name="custom-validator-file"></a>
  When a custom validation is used more than once or the validation is some
  regular expression mapping, create a custom validator file.
  <sup>[[ссылка](#custom-validator-file)]</sup>

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

* <a name="app-validators"></a>
  Храните файлы определенных вами валидаторов в `app/validators`.
  <sup>[[ссылка](#app-validators)]</sup>

* <a name="custom-validators-gem"></a>
  Подумайте о том, чтобы выделить ряд определенных вами валидаторов в отдельный
  гем, если вы работаете над рядом схожих приложений и валидаторы имеют
  достаточно обобщенные функции.
  <sup>[[ссылка](#custom-validators-gem)]</sup>

* <a name="named-scopes"></a>
  Use named scopes freely.
  <sup>[[ссылка](#named-scopes)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    scope :active, -> { where(active: true) }
    scope :inactive, -> { where(active: false) }

    scope :with_orders, -> { joins(:orders).select('distinct(users.id)') }
  end
  ```

* <a name="named-scope-class"></a>
  When a named scope defined with a lambda and parameters becomes too
  complicated, it is preferable to make a class method instead which serves the
  same purpose of the named scope and returns an `ActiveRecord::Relation`
  object. Arguably you can define even simpler scopes like this.
  <sup>[[ссылка](#named-scope-class)]</sup>

  ```Ruby
  class User < ActiveRecord::Base
    def self.with_orders
      joins(:orders).select('distinct(users.id)')
    end
  end
  ```

  Note that this style of scoping can not be chained in the same way as named scopes. For instance:

  ```Ruby
  # unchainable
  class User < ActiveRecord::Base
    def User.old
      where('age > ?', 80)
    end

    def User.heavy
      where('weight > ?', 200)
    end
  end
  ```

  In this style both `old` and `heavy` work individually, but you can not call
  `User.old.heavy`, to chain these scopes use:

  ```Ruby
  # chainable
  class User < ActiveRecord::Base
    scope :old, -> { where('age > 60') }
    scope :heavy, -> { where('weight > 200') }
  end
  ```

* <a name="beware-update-attribute"></a>
  Beware of the behavior of the [`update_attribute`
  ](http://api.rubyonrails.org/classes/ActiveRecord/Persistence.html#method-i-update_attribute)
  method. It doesn't  run the model validations (unlike `update_attributes`)
  and could easily corrupt the model state.
  <sup>[[ссылка](#beware-update-attribute)]</sup>

* <a name="user-friendly-urls"></a>
  Use user-friendly URLs. Show some descriptive attribute of the model
  in the URL rather than its `id`.  There is more than one way to achieve this:
  <sup>[[ссылка](#user-friendly-urls)]</sup>

  * Override the `to_param` method of the model. This method is used by Rails
    for constructing a URL to the object. The default implementation returns the
    `id` of the record as a String. It could be overridden to include another
    human-readable attribute.

        ```Ruby
        class Person
          def to_param
            "#{id} #{name}".parameterize
          end
        end
        ```

    In order to convert this to a URL-friendly value, `parameterize` should be
    called on the string. The `id` of the object needs to be at the beginning so
    that it can be found by the `find` method of ActiveRecord.

  * Use the `friendly_id` gem. It allows creation of human-readable URLs by using
    some descriptive attribute of the model instead of its `id`.

        ```Ruby
        class Person
          extend FriendlyId
          friendly_id :name, use: :slugged
        end
        ```

  Check the [gem documentation](https://github.com/norman/friendly_id)
  for more information about its usage.

* <a name="find-each"></a>
  Use `find_each` to iterate over a collection of AR objects. Looping through
  a collection of records from the database (using the `all`
  method, for example) is very inefficient since it will try to
  instantiate all the objects at once. In that case, batch processing
  methods allow you to work with the records in batches, thereby
  greatly reducing memory consumption.
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
  Since [Rails creates callbacks for dependent
  associations](https://github.com/rails/rails/issues/3458), always call
  `before_destroy` callbacks that perform validation with `prepend: true`.
  <sup>[[ссылка](#before_destroy)]</sup>

  ```Ruby
  # плохо (roles will be deleted automatically even if super_admin? is true)
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
### Запросы ActiveRecord

* <a name="avoid-interpolation"></a>
  Avoid string interpolation in
  queries, as it will make your code susceptible to SQL injection
  attacks.
  <sup>[[ссылка](#avoid-interpolation)]</sup>

  ```Ruby
  # плохо - param will be interpolated unescaped
  Client.where("orders_count = #{params[:orders]}")

  # хорошо - param will be properly escaped
  Client.where('orders_count = ?', params[:orders])
  ```

* <a name="named-placeholder"></a>
  Consider using named placeholders instead of positional placeholders
  when you have more than 1 placeholder in your query.
  <sup>[[ссылка](#named-placeholder)]</sup>

  ```Ruby
  # okish
  Client.where(
    'created_at >= ? AND created_at <= ?',
    params[:start_date], params[:end_date]
  )

  # хорошо
  Client.where(
    'created_at >= :start_date AND created_at <= :end_date',
    start_date: params[:start_date], end_date: params[:end_date]
  )
  ```

* <a name="find"></a>
  Favor the use of `find` over `where`
  when you need to retrieve a single record by id.
  <sup>[[ссылка](#find)]</sup>

  ```Ruby
  # плохо
  User.where(id: id).take

  # хорошо
  User.find(id)
  ```

* <a name="find_by"></a>
  Favor the use of `find_by` over `where`
  when you need to retrieve a single record by some attributes.
  <sup>[[ссылка](#find_by)]</sup>

  ```Ruby
  # плохо
  User.where(first_name: 'Bruce', last_name: 'Wayne').first

  # хорошо
  User.find_by(first_name: 'Bruce', last_name: 'Wayne')
  ```

* <a name="find_each"></a>
  Use `find_each` when you need to process a lot of records.
  <sup>[[ссылка](#find_each)]</sup>

  ```Ruby
  # плохо - loads all the records at once
  # This is very inefficient when the users table has thousands of rows.
  User.all.each do |user|
    NewsMailer.weekly(user).deliver_now
  end

  # хорошо - records are retrieved in batches
  User.find_each do |user|
    NewsMailer.weekly(user).deliver_now
  end
  ```

* <a name="where-not"></a>
  Favor the use of `where.not` over SQL.
  <sup>[[ссылка](#where-not)]</sup>

  ```Ruby
  # плохо
  User.where("id != ?", id)

  # хорошо
  User.where.not(id: id)
  ```

## Миграции

* <a name="schema-version"></a>
  Храните файл `schema.rb` (или `structure.sql`) в вашей системе управления
  версиями.
  <sup>[[ссылка](#schema-version)]</sup>

* <a name="db-schema-load"></a>
  Используйте вызов `rake db:schema:load` вместо `rake db:migrate` для
  инициализации пустой базы данных.
  <sup>[[ссылка](#db-schema-load)]</sup>

* <a name="default-migration-values"></a>
  Enforce default values in the migrations themselves instead of in the
  application layer.
  <sup>[[ссылка](#default-migration-values)]</sup>

  ```Ruby
  # плохо - application enforced default value
  def amount
    self[:amount] or 0
  end
  ```

  While enforcing table defaults only in Rails is suggested by many
  Rails developers, it's an extremely brittle approach that
  leaves your data vulnerable to many application bugs.  And you'll
  have to consider the fact that most non-trivial apps share a
  database with other applications, so imposing data integrity from
  the Rails app is impossible.

* <a name="foreign-key-constraints"></a>
  Enforce foreign-key constraints. As of Rails 4.2, ActiveRecord
  supports foreign key constraints natively.
  <sup>[[ссылка](#foreign-key-constraints)]</sup>

* <a name="change-vs-up-down"></a>
  When writing constructive migrations (adding tables or columns),
  use the `change` method instead of `up` and `down` methods.
  <sup>[[ссылка](#change-vs-up-down)]</sup>

  ```Ruby
  # the old way
  class AddNameToPeople < ActiveRecord::Migration
    def up
      add_column :people, :name, :string
    end

    def down
      remove_column :people, :name
    end
  end

  # the new prefered way
  class AddNameToPeople < ActiveRecord::Migration
    def change
      add_column :people, :name, :string
    end
  end
  ```

* <a name="no-model-class-migrations"></a>
  Don't use model classes in migrations. The model classes are constantly
  evolving and at some point in the future migrations that used to work might
  stop, because of changes in the models used.
<sup>[[ссылка](#no-model-class-migrations)]</sup>

## Представления

* <a name="no-direct-model-view"></a>
  Ни при каких условиях не следует работать с моделями напрямую из отображений.
  <sup>[[ссылка](#no-direct-model-view)]</sup>

* <a name="no-complex-view-formatting"></a>
  Избегайте сложной логики в отображениях, выделяйте этот код во вспомогательные
  методы отображений или выносите их в модель.
  <sup>[[ссылка](#no-complex-view-formatting)]</sup>

* <a name="partials"></a>
  Избегайте повторений кода, используйте отдельные шаблоны и подшаблоны.
  <sup>[[ссылка](#partials)]</sup>

## Интернационализация

* <a name="locale-texts"></a>
  Строки и другие локальные настройки и детали следует выносить из представлений
  и контроллеров, а также моделей в файлы для конкретных локалей в директории
  `config/locales`.
  <sup>[[ссылка](#locale-texts)]</sup>

* <a name="translated-labels"></a>
  Когда вам нужно перевести идентификаторы для моделей ActiveRecord, применяйте
  контекст `activerecord`:
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

  В этом случае `User.model_name.human` вернет `'Member'` и
  `User.human_attribute_name('name')` вернет `'Full name'`. Переводы этих
  атрибутов будут использоваться в качестве идентификаторов в представлениях.

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
  # плохо
  I18n.t :record_invalid, :scope => [:activerecord, :errors, :messages]

  # хорошо
  I18n.t 'activerecord.errors.messages.record_invalid'
  ```

* <a name="i18n-guides"></a>
  Более подробную информацию по интернационализации (I18n) в Rails можно найти
  по адресу [API интернационализации Rails
  ](http://rusrails.ru/rails-internationalization-i18n-api) либо
  [Rails Guides](http://guides.rubyonrails.org/i18n.html) (английский оригинал).
<sup>[[ссылка](#i18n-guides)]</sup>

## Ресурсы

Применяйте
[конвейер ресурсов](http://guides.rubyonrails.org/asset_pipeline.html)
(assets pipeline) для упорядочения структуры вашего приложения.

* <a name="reserve-app-assets"></a>
  Зарезервируйте директорию `app/assets` для ваших собственных таблиц стилей,
  скриптов и/или изображений.
  <sup>[[ссылка](#reserve-app-assets)]</sup>

* <a name="lib-assets"></a>
  Используйте директорию `lib/assets` для ваших собственных библиотек, которые
  не реализуют первичный функционал вашего приложения.
  <sup>[[ссылка](#lib-assets)]</sup>

* <a name="vendor-assets"></a>
  Код сторонних библиотек, например, [jQuery](http://jquery.com/) или
  [bootstrap](http://twitter.github.com/bootstrap/), следует размещать в
  директории `vendor/assets`.
  <sup>[[ссылка](#vendor-assets)]</sup>

* <a name="gem-assets"></a>
  По возможности используйте упакованные версии необохдимых ресурсов, например:
  <sup>[[ссылка](#gem-assets)]</sup>

  - [jquery-rails](https://github.com/rails/jquery-rails);
  - [jquery-ui-rails](https://github.com/joliss/jquery-ui-rails);
  - [bootstrap-sass](https://github.com/thomas-mcdonald/bootstrap-sass);
  - [zurb-foundation](https://github.com/zurb/foundation).

## Почтовые модули

* <a name="mailer-name"></a>
  Называйте почтовые модули по образцу `SomethingMailer`. Без суффикса `Mailer`
  не сразу будет понятно, что это почтовый модуль и какие представления
  связаны с этим модулем.
  <sup>[[ссылка](#mailer-name)]</sup>

* <a name="html-plain-email"></a>
  Создавайте шаблоны представлений в текстовом формате и в форате HTML.
  <sup>[[ссылка](#html-plain-email)]</sup>

* <a name="enable-delivery-errors"></a>
  Включите вызов ошибок при проблемах с доставкой почты в вашем окружении для
  разработки. По умолчанию эти вызовы отключены.
  <sup>[[ссылка](#enable-delivery-errors)]</sup>

  ```Ruby
  # config/environments/development.rb

  config.action_mailer.raise_delivery_errors = true
  ```

* <a name="local-smtp"></a>
  Используйте локальный сервер SMTP, например, [Mailcatcher
  ](https://github.com/sj26/mailcatcher) в вашем окружении разработки.
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
  <%= link_to 'here', course_path(@course) %>

  # хорошо
  You can always find more info about this course
  <%= link_to 'here', course_url(@course) %>
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
  Методом доставки почты для разработки и развертывания должен быть `smtp`:
  <sup>[[ссылка](#delivery-method-smtp)]</sup>

  ```Ruby
  # config/environments/development.rb, config/environments/production.rb

  config.action_mailer.delivery_method = :smtp
  ```

* <a name="inline-email-styles"></a>
  При рассылке электронной почты в формате HTML все описания стилей должны быть
  включены в текст, потому что некоторые почтовые программы неверно обрабатывают
  внешние таблицы стилей. Однако, это приводит к сложностям в поддержке таких
  таблиц и повторениям в коде. Для преобразования и внедрения стилей в текст
  письма существую два схожим по функциональности гема:
  [premailer-rails](https://github.com/fphilipe/premailer-rails) и
  [roadie](https://github.com/Mange/roadie).
  <sup>[[ссылка](#inline-email-styles)]</sup>

* <a name="background-email"></a>
  Избегайте рассылки почты параллельно к генерации страницы в ответ на запрос
  пользователя. Это вызовет задержки в загрузке страницы, и запрос может быть
  отклонен из-за превышения таймаута, если вы рассылаете много писем. Вы можете
  преодолеть данное ограничение, вызывая процессы рассылки в фоне, например, при
  помощи гема [sidekiq](https://github.com/mperham/sidekiq).
  <sup>[[ссылка](#background-email)]</sup>

## Время

* <a name="tz-config"></a>
  Config your timezone accordingly in `application.rb`.
  <sup>[[ссылка](#time-now)]</sup>

  ```Ruby
  config.time_zone = 'Eastern European Time'
  # optional - note it can be only :utc or :local (default is :utc)
  config.active_record.default_timezone = :local
  ```

* <a name="time-parse"></a>
  Don't use `Time.parse`.
  <sup>[[ссылка](#time-parse)]</sup>

  ```Ruby
  # плохо
  Time.parse('2015-03-02 19:05:37') # => Will assume time string given is in the system's time zone.

  # хорошо
  Time.zone.parse('2015-03-02 19:05:37') # => Mon, 02 Mar 2015 19:05:37 EET +02:00
  ```

* <a name="time-now"></a>
  Don't use `Time.now`.
  <sup>[[ссылка](#time-now)]</sup>

  ```Ruby
  # плохо
  Time.now # => Returns system time and ignores your configured time zone.

  # хорошо
  Time.zone.now # => Fri, 12 Mar 2014 22:04:47 EET +02:00
  Time.current # Same thing but shorter.
  ```

## Bundler

* <a name="dev-test-gems"></a>
  Держите гемы, которые используются только для разработки и/или тестирования в
  соответствующих группах вашего `Gemfile`.
  <sup>[[ссылка](#dev-test-gems)]</sup>

* <a name="only-good-gems"></a>
  Применяйте в своих проектах только рекомендованные временем библиотеки. Если
  вы задумываетесь, не включить ли в проект малоизвестный гем, вам следует
  сперва внимательно просмотреть его исходных код.
  <sup>[[ссылка](#only-good-gems)]</sup>

* <a name="os-specific-gemfile-locks"></a>
  Системозависимые библиотеки в вашем проекте будут причиной постоянного
  изменения файла `Gemfile.lock` для проектов с несколькими разработчиками,
  работающими на разных операционных системах. Поэтому внесите все библиотеки,
  написанные для `OS X`, в группу `darwin`, а библиотеки, написанные
  для `Linux`, в группу `linux`.
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

  Для включения нужных библиотек только в нужном окружении, добавьте в файл
  `config/application.rb` следующие строки:

  ```Ruby
  platform = RUBY_PLATFORM.match(/(linux|darwin)/)[0].to_sym
  Bundler.require(platform)
  ```

* <a name="gemfile-lock"></a>
  Сохраняйте файл `Gemfile.lock` в вашей системе управления версиями. Этот файл
  содержит неслучайную информацию, так вы сможете удостовериться, что все члены
  вашей команды установят точно ту же версию библиотек, что и вы, при помощи
  команды `bundle install`.
  <sup>[[ссылка](#gemfile-lock)]</sup>

## Гемы с дефектами

В настоящем списке перечислены гемы, которые либо имеют проблемы в реализации,
либо имеют лучшие аналоги. Вам не следует применять их в своих проектах.

* [rmagick](http://rmagick.rubyforge.org/) - this gem is notorious
  for its memory consumption.
  Use [minimagick](https://github.com/probablycorey/mini_magick) instead.


* [autotest](http://www.zenspider.com/ZSS/Products/ZenTest/) - old solution
  for running tests automatically. Far  inferior to
  [guard](https://github.com/guard/guard) and
  [watchr](https://github.com/mynyml/watchr).


* [rcov](https://github.com/relevance/rcov) - code coverage tool, not compatible
  with Ruby 1.9. Use [SimpleCov](https://github.com/colszowka/simplecov) instead.


* [therubyracer](https://github.com/cowboyd/therubyracer) - the use of this gem
  in production is strongly discouraged as it uses a very large amount
  of memory. I'd suggest using `node.js` instead.


Работа над этим списком лишь начата. Пожалуйста, если вы знаете о популярных,
но дефектных библиотеках, то проинформируйте меня.

## Управление процессами

*  <a name="foreman"></a>
   Если в вашем проекте есть много зависимостей от внешних процессов, применяйте
   библиотеку [foreman](https://github.com/ddollar/foreman) для управления ими.
   <sup>[[ссылка](#foreman)]</sup>

# Дополнительные источники

Существует несколько отличных источников по стилю офрмления приложений на Rails,
на которые вы можете взглянуть, если у вас будет свободное время:

<!--- @FIXME: Find russian translations. -->
* [The Rails 4 Way](http://www.amazon.com/The-Rails-Addison-Wesley-Professional-Ruby/dp/0321944275)
* [Ruby on Rails Guides](http://guides.rubyonrails.org/)
* [The RSpec Book](http://pragprog.com/book/achbd/the-rspec-book)
* [The Cucumber Book](http://pragprog.com/book/hwcuc/the-cucumber-book)
* [Everyday Rails Testing with RSpec](https://leanpub.com/everydayrailsrspec)
* [Better Specs for RSpec](http://betterspecs.org)

# Сотрудничество

Ничто, описанное в этом руководстве, не высечено в камне. И я очень хотел бы
сотрудничать со всеми, кто интересуется стилистикой оформления кода Rails,
чтобы мы смогли вместе создать ресурс, который был бы полезен для всего сообщества
программистов на Руби.

Не стесняйтесь создавать отчеты об ошибках и присылать мне запросы на интеграцию
вашего кода. И заранее большое спасибо за вашу помощь!

Вы можете поддержать проект (и РубоКоп) денежным взносом при помощи
[gittip](https://www.gittip.com/bbatsov).

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
[transmuter]: https://github.com/TechnoGate/transmuter