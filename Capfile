load 'deploy'
Dir['vendor/plugins/*/recipes/*.rb'].each { |plugin| load(plugin) }
Dir['lib/recipes/*.rb'].each { |recipe| load(recipe) }

set :host, 'ec2-174-129-142-226.compute-1.amazonaws.com'
set :application, "testapp"
set :scm, :git
set :repository,  "git://github.com/mitya/testapp.git"
set :deploy_via, :remote_cache
set :deploy_to, "/var/apps/#{application}"
set :user, "root"
set :ssh_options, {:keys => ["#{ENV["HOME"]}/.ssh/id_demo"]}
set :database, 'testapp'

server host, :web, :app, :db, :primary => true

namespace :deploy do
  desc "Restarts mod_rails with restart.txt"
  task :restart, :roles => :app, :except => { :no_release => true } do
    run "touch #{current_path}/tmp/restart.txt"
  end
  
  desc "Configures passenger for the app"
  task :configure_passenger, :role => :app do 
    put <<-CONFIG, "/etc/apache2/sites-available/#{application}"
      <VirtualHost #{host}:80>
        ServerName #{host}
        DocumentRoot "/var/apps/#{application}/current/public" 
        RailsEnv production
        <directory "/var/apps/#{application}/current/public">
          Order allow,deny
          Allow from all
        </directory>
      </VirtualHost>
    CONFIG
    sudo "ln -s /etc/apache2/sites-available/#{application} /etc/apache2/sites-enabled/#{application}"
  end
  
  desc "Create production database"
  task :create_db, :roles => :db do
    sudo "createdb #{database}"
  end
  
  desc "Start is no-op with passenger"
  task :start, :roles => :app do
  end
  
  desc "Stop is no-op with passenger"
  task :stop, :roles => :app do
  end
  
  task :fix_permissions, :roles => :app do
    sudo "chown www-data:www-data #{current_path}/config/environment.rb"
  end
  
  task :restart_apache, :role => :web do
    sudo "/etc/init.d/apache2 restart"
  end
end

after 'deploy:setup', 'deploy:configure_passenger'
after 'deploy:symlink', 'deploy:fix_permissions'
after 'deploy:cold', 'deploy:restart_apache'
