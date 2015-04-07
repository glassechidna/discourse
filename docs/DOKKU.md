This fork of [Discourse](http://www.discourse.org/) is intended to be fully compatible with deploying to Dokku.

## Installing locally

1. `$ git clone git@github.com:rwdaigle/discourse.git && cd discourse`
1. `$ bundle install`
1. `$ cp .env.sample .env`
1. `$ bundle exec rake db:create db:migrate`
1. `$ foreman start web`
1. Open [http://localhost:5000](http://localhost:5000) to see Discourse running locally

## Deploying to Dokku

### Prerequisites

[Dokku](https://github.com/progrium/dokku)... obviously.

### Dokku Plugins

We need a postgresql plugin and a redis plugin.

    cd /var/lib/dokku/plugins
    sudo git clone https://github.com/Kloadut/dokku-pg-plugin
    sudo git clone https://github.com/jezdez/dokku-redis-plugin
    sudo dokku plugins-install

### Deploy

Deploy as you would any other Dokku app:

    git remote add deploy dokku@yourhostname:discourse
    git push deploy master

The first deployment will fail as we haven't setup a database etc. or specified
some necessary ENV vars. After deployment fails, login into the dokku host and
execute:

    sudo dokku postgresql:create discourse
    sudo dokku redis:create discourse
    sudo dokku config:set discourse SECRET_TOKEN=$(openssl rand -base64 32)
    sudo dokku config:set discourse NEW_RELIC_APP_NAME=Discourse

then deploy again.

__Note:__ Technically all these ENV variables should be able to be set in one command. However, at the time of writing, setting multiple ENV vars in one go was not working reliably.

## Configure Discourse

Once the app is deployed you still need to establish the admin user and set some basic settings.

1. Log into the app, using your preferred auth provider.
1. In order to make the first user an Admin, SSH into the Dokku host and execute `$ sudo dokku run discourse rails console`
1. Enter the following commands.

```ruby
u = User.first
u.admin = true
u.approved = true
u.save
```

4. Set `force_hostname` to your applications domain.

    This step is required for Discourse to properly form links sent with account confirmation emails and password resets. The auto detected application url would point to an Amazon AWS instance.

    Since you can't log in yet, you can set `force_hostname` in the console.

```ruby
SiteSetting.create(:name => 'force_hostname', :data_type =>1, :value=>'<your hostname>')
```
