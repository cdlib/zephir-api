source "https://rubygems.org"

ruby '2.5.7'
 
gem "sinatra", :require => 'sinatra/base'
gem "sinatra-activerecord"
gem "sinatra-contrib", :require => 'sinatra/contrib/all'
gem "activerecord", "~> 6.1.7.1"
gem "mysql2"
gem "rdiscount" #markdown support
gem "yard-sinatra", :require => false # document generation

group :test do
  gem 'rack-test', :require => 'rack/test'
  gem 'minitest'
  gem "sqlite3", "1.3.10" # test database
end
