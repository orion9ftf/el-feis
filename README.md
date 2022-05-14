# Autenticación con Facebook

> Necesitamos dos cosas:
  - Primero que haya una interfás donde el usuario quiera registrarse.
  - Segundo, un sistema de registro, cómo hacemos eso:

> Revisemos nuestras versiones de Ruby y Rails para trabajar:

- Ruby -v 2.5.3
- Rails -v 5.2.5

No olvides versionar, generar las variables de entorno para la autenticación y para todo lo que pueda llevar contraseñas, emails, api_key, etc.(esta de más decirlo no?).

***Gemas a utilizar:***

Para devise, en caso que te genere conflictos al momento de la autenticación debes poner la siguiente exxtención al lado de la gem, github: 'heartcombo/devise'

` gem 'devise', github: 'heartcombo/devise'`

Esta gema nos permitirá hacer magia, y hacerle la vida más sencilla a nuestros señores usuarios:

` gem 'omniauth-facebook'`

Ambas gemas nos permite autenticar con proveedores externos: 

`gem 'omniauth'`

Esta es en caso de que omniauth nos genere conflictos de seguridad al momento de autenticar a nuestro usuario:

` gem 'omniauth-rails_csrf_protection', '~> 1.0'`

### Generemos nuestras tablas:

Para darle vida a todo esto haremos la siguente app:

` $ rails new feis -d postgresql`

**Haremos la app con postgresql pata que nuestros datos de prueba puedan persistir.**

No olvides ingresar a la app mediante el comando: `$ cd [NOMBRE_APP]`

Crearemos la tabla post:

` $ rails g scaffold post name content `

Primero revisar si la migración esta en orden o necesitamos agreagar otro campo, si todo esta bien, migramos:

` $ rails db:create db:migrate`

 `db:create`: es para crear la base de datos en nuestra app, y como Ruby es tan inteligente, reconoce el comando de inmediato.

**Generamos el modelo User con devise**

Recuerda la instalación de la gema:

`$ bundle install`

`$ rails g devise:install`

**Configurar la instalación es de suma importancia.**

Generamos el modelo: 

`$ rails g devise User`

Vamos a necesitar la configuración de las vistas asi que las instalamos de la siguiente manera:

`$ rails g devise:views User`

**Esto nos permitirá hacer la autenticación con facebook.**

### Adaptemos el código:

El modelo user.rb nos debe quedar de la siguiente manera:

```
class User < ApplicationRecord
  devise :database_authenticatable, :registerable,
         :recoverable, :rememberable, :validatable, - :omniauthable, omniauth_providers: [:facebook]
  
  def self.from_omniauth(auth)
    where(provider: auth.provider, uid: auth.uid).- first_or_create do |user|
      user.email = auth.info.email
      user.name = auth.info.name # en caso de tener el campo de nombre en devise user
      user.password = Devise.friendly_token[0, 20]
      end
  end
end
```


### Rutas:

routes.rb

```
Rails.application.routes.draw do
  devise_for :users, controllers: {
    omniauth_callbacks: 'users/omniauth_callbacks'
  }
  resources :posts

  root 'posts#index'
end
```


config/initializers/devise.rb

En la línea 274 aprox.

```
config.omniauth :facebook, ENV['FACEBOOK_CLIENT_ID'], ENV['FACEBOOK_SECRET_ID']
```


**DATO: generar varibles de entorno nos entrega una muy buena práctica de nuestro trabajo, ya que evitamos que nuestras credenciales queden expuestas y con riesgo de subirlas a repositorios públicos o la nube(aws por ejemplo).**


