# Ruby 2.7 to 3.4.7 Upgrade Guide (at Graviton EC2 instance)

This guide documents the complete process of upgrading a Rails application from Ruby 2.7 to Ruby 3.4.7 on Amazon Linux 2023 (ARM64/Graviton).

## Performance Results

**Before (Ruby 2.7):**
- Dashboard response time: 1500-7700ms (avg ~2000ms)
- High variance and inconsistent performance

**After (Ruby 3.4.7):**
- Dashboard response time: 218-457ms (avg ~245ms)
- **6-8x faster** with much more consistent performance
- **94% improvement** in worst-case scenarios

---

## Prerequisites

- Amazon Linux 2023 instance (ARM64/Graviton recommended)
- Existing Rails application running Ruby 2.7
- PostgreSQL database (RDS or local)
- SSH/SSM access to the instance

---

## Step 0: Install System Dependencies

```bash
sudo dnf install -y gcc make openssl-devel zlib-devel readline-devel \
  libffi-devel libyaml-devel bzip2 gcc-c++ postgresql-devel git
```

---

## Step 1: Install safecastapi

```bash
git clone https://github.com/Safecast/safecastapi.git
```

---

## Step 2: Install rbenv and ruby-build

```bash
# Clone rbenv
git clone https://github.com/rbenv/rbenv.git ~/.rbenv

# Clone ruby-build plugin
git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build

# Add to shell
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
source ~/.bashrc
```

---

## Step 3: Install Ruby 3.4.7

```bash
# install 3.4.7
rbenv install 3.4.7

# run global
rbenv global 3.4.7

# Verify installation
ruby -v
# Should output: ruby 3.4.7 (2025-10-08 revision 7a5688e2a2) +PRISM [aarch64-linux]
```

---

## Step 4: Update Application Configuration

### Update .ruby-version
```bash
cd ~/your-app
echo "3.4.7" > .ruby-version
```

### Update Gemfile
```ruby
# Change Ruby version
sed -i "s/ruby '[0-9.]*'/ruby '3.4.7'/" Gemfile
```

---

## Step 5: Add Missing Standard Library Gems

Ruby 3.4 removed several gems from the default bundle. Add these to your Gemfile:

```ruby
cat >> Gemfile << 'EOF'

# add additional gems for Ruby 3.4.x
gem 'mutex_m'
gem 'ostruct'
gem 'bigdecimal'
gem 'drb'
gem 'logger'
gem 'csv'
gem 'multi_json', '~> 1.15'
EOF
```

---

## Step 6: Upgrade Rails

Ruby 3.4 requires Rails 7.0+. Update your Gemfile:

```bash
# Update Rails version
sed -i "s/gem 'rails', '~> 6.0.*/gem 'rails', '~> 7.1.0'/" Gemfile
```

---

## Step 7: Update Incompatible Gems

### activerecord-postgis-adapter
```bash
# Update to version compatible with Rails 7
sed -i "s/gem 'activerecord-postgis-adapter', '~> 6.0'/gem 'activerecord-postgis-adapter', '~> 9.0'/" Gemfile
```

### aws-sdk-ses (if using SES)
```bash
echo "gem 'aws-sdk-ses'" >> Gemfile
```

---

## Step 8: Install Dependencies

clear dependency conflicts:
```bash
bundle update
```
update output logs
```text
$ bundle update
[DEPRECATED] Platform :mingw, :mswin, :x64_mingw is deprecated. Please use platform :windows instead.
[DEPRECATED] Found x64-mingw32 in lockfile, which is deprecated. Using x64-mingw-ucrt, the replacement for x64-mingw32 in modern rubies, instead. Support for x64-mingw32 will be removed in Bundler 4.0.
Fetching gem metadata from https://rubygems.org/.........
Resolving dependencies...
Using rake 13.3.1 (was 13.0.6)
Using concurrent-ruby 1.3.5 (was 1.2.2)
Using minitest 5.26.1 (was 5.18.0)
Using builder 3.3.0 (was 3.2.4)
Using erubi 1.13.1 (was 1.12.0)
Using mini_portile2 2.8.9 (was 2.8.1)
Using racc 1.8.1 (was 1.6.2)
Using rack 2.2.21 (was 2.2.6.4)
Using nio4r 2.7.5 (was 2.5.9)
Using zeitwerk 2.7.3 (was 2.6.7)
Using timeout 0.4.4 (was 0.3.1)
Using marcel 1.0.4 (was 1.0.2)
Using mini_mime 1.1.5 (was 1.1.2)
Using date 3.5.0 (was 3.3.3)
Using rgeo 3.0.1 (was 2.4.0)
Using public_suffix 6.0.2 (was 5.0.1)
Using ast 2.4.3 (was 2.4.2)
Using execjs 2.10.0 (was 2.7.0)
Using aws-eventstream 1.4.0 (was 1.2.0)
Using aws-partitions 1.1181.0 (was 1.714.0)
Using thor 1.4.0 (was 1.2.1)
Using bcrypt 3.1.20 (was 3.1.18)
Using msgpack 1.8.0 (was 1.6.0)
Using ffi 1.17.2 (was 1.15.5)
Using byebug 12.0.0 (was 11.1.3)
Using matrix 0.4.3 (was 0.4.2)
Using regexp_parser 2.11.3 (was 2.8.0)
Using ssrf_filter 1.3.0 (was 1.1.1)
Using chartkick 5.2.1 (was 3.4.2)
Using rexml 3.4.4 (was 3.2.5)
Using diff-lcs 1.6.2 (was 1.5.0)
Using dotenv 3.1.8 (was 2.8.1)
Using multi_json 1.17.0 (was 1.15.0)
Using fabrication 3.0.0 (was 2.30.0)
Using mime-types-data 3.2025.0924 (was 3.2022.0105)
Using hashdiff 1.2.1 (was 1.0.1)
Using method_source 1.1.0 (was 1.0.0)
Using newrelic_rpm 9.22.0 (was 8.15.0)
Using parallel 1.27.0 (was 1.23.0)
Using rspec-support 3.13.6 (was 3.11.1)
Using rubyzip 3.2.2 (was 2.3.2)
Using tilt 2.6.1 (was 2.0.11)
Using spring 4.4.0 (was 4.1.0)
Using i18n 1.14.7 (was 1.12.0)
Using tzinfo 2.0.6 (was 1.2.11)
Using mini_magick 5.3.1 (was 4.12.0)
Using excon 1.3.1 (was 0.93.0)
Using nokogiri 1.18.10 (was 1.14.3)
Using rack-test 2.2.0 (was 2.0.2)
Using request_store 1.7.0 (was 1.5.1)
Using sprockets 3.7.5 (was 3.7.2)
Using websocket-driver 0.8.0 (was 0.7.5)
Using puma 7.1.0 (was 6.4.0)
Using net-protocol 0.2.2 (was 0.2.1)
Using addressable 2.8.7 (was 2.8.1)
Using parser 3.3.10.0 (was 3.2.2.1)
Using autoprefixer-rails 10.4.21.0 (was 10.2.4.0)
Using uglifier 4.2.1 (was 4.2.0)
Using aws-sigv4 1.12.1 (was 1.5.2)
Using bootsnap 1.18.6 (was 1.15.0)
Using ruby-vips 2.2.5 (was 2.1.4)
Using rb-inotify 0.11.1 (was 0.10.1)
Using crack 1.0.1 (was 0.4.5)
Using elasticsearch-api 8.19.2 (was 7.17.1)
Using mime-types 3.7.0 (was 3.4.1)
Using pry 0.15.2 (was 0.14.1)
Using rspec-core 3.13.6 (was 3.11.0)
Using rspec-expectations 3.13.5 (was 3.11.0)
Using rspec-mocks 3.13.7 (was 3.11.1)
Using unicode-display_width 3.2.0 (was 2.4.2)
Using activesupport 7.1.6 (was 6.0.6.1)
Using loofah 2.24.1 (was 2.19.1)
Using net-imap 0.5.12 (was 0.3.4)
Using net-smtp 0.5.1 (was 0.3.3)
Using launchy 3.1.1 (was 2.5.0)
Using rubocop-ast 1.48.0 (was 1.28.1)
Using aws-sdk-core 3.236.0 (was 3.170.0)
Using formatador 1.2.2 (was 1.1.0)
Using image_processing 1.14.0 (was 1.12.2)
Using listen 3.9.0 (was 3.8.0)
Using webmock 3.26.1 (was 3.18.1)
Using faraday-net_http 3.4.2 (was 1.0.1)
Using pry-byebug 3.11.0 (was 3.10.1)
Using pry-rails 0.3.11 (was 0.3.9)
Using rspec-its 2.0.0 (was 1.3.0)
Using rails-dom-testing 2.3.0 (was 2.0.3)
Using globalid 1.3.0 (was 1.0.1)
Using activemodel 7.1.6 (was 6.0.6.1)
Using delayed_job 4.1.13 (was 4.1.11)
Using rails-html-sanitizer 1.6.2 (was 1.5.0)
Using capybara 3.40.0 (was 3.38.0)
Using mail 2.9.0 (was 2.8.0.1)
Using rubocop 1.81.7 (was 1.51.0)
Using aws-sdk-elasticbeanstalk 1.95.0 (was 1.52.0)
Using aws-sdk-kms 1.117.0 (was 1.61.0)
Using fog-core 2.6.0 (was 2.3.0)
Using faraday 2.14.0 (was 1.10.1)
Using activejob 7.1.6 (was 6.0.6.1)
Using activerecord 7.1.6 (was 6.0.6.1)
Using activemodel-serializers-xml 1.0.3 (was 1.0.2)
Using carrierwave 2.2.6 (was 2.2.3)
Using actionview 7.1.6 (was 6.0.6.1)
Using email_spec 2.3.0 (was 2.2.0)
Using rubocop-performance 1.26.1 (was 1.16.0)
Using rubocop-rails 2.33.4 (was 2.19.1)
Using rubocop-rspec 3.7.0 (was 2.17.0)
Using aws-sdk-s3 1.203.0 (was 1.117.2)
Using fog-xml 0.1.5 (was 0.1.4)
Using rgeo-activerecord 7.0.1 (was 6.2.2)
Using database_cleaner-active_record 2.2.2 (was 2.0.1)
Using delayed_job_active_record 4.1.11 (was 4.1.7)
Using actionpack 7.1.6 (was 6.0.6.1)
Using jbuilder 2.14.1 (was 2.11.5)
Using elasticsearch 8.19.2 (was 7.17.1)
Using activerecord-postgis-adapter 9.0.2 (was 6.0.3)
Using database_cleaner 2.1.0 (was 2.0.1)
Using actioncable 7.1.6 (was 6.0.6.1)
Using activestorage 7.1.6 (was 6.0.6.1)
Using actionmailer 7.1.6 (was 6.0.6.1)
Using railties 7.1.6 (was 6.0.6.1)
Using draper 4.0.4 (was 4.0.2)
Using has_scope 0.8.2 (was 0.8.0)
Using sprockets-rails 3.5.2 (was 3.4.2)
Using simple_form 5.4.0 (was 5.1.0)
Using elasticsearch-model 8.0.1 (was 7.2.1)
Using actionmailbox 7.1.6 (was 6.0.6.1)
Using actiontext 7.1.6 (was 6.0.6.1)
Using aws-sdk-rails 5.1.0 (was 3.7.1)
Using responders 3.2.0 (was 3.0.1)
Using dotenv-rails 3.1.8 (was 2.8.1)
Using jquery-rails 4.6.1 (was 4.5.1)
Using lograge 0.14.0 (was 0.12.0)
Using turbo-rails 1.5.0 (was 1.1.1)
Using rspec-rails 7.1.1 (was 5.1.2)
Using rails 7.1.6 (was 6.0.6.1)
Using devise 4.9.4 (was 4.8.1)
Using devise-i18n 1.15.0 (was 1.10.2)
Installing pg 1.6.2 (was 1.4.5) with native extensions
Bundle updated!
```

bundle install
```bash
gem install bundler
gem update --system 3.7.2
bundle install
```

---

## Step 9: Fix Breaking Changes

### Mailer Configuration (if using AWS SES)

Edit `config/initializers/mailer.rb`:

**Old (Ruby 2.7):**
```ruby
Aws::Rails.add_action_mailer_delivery_method(:aws_sdk, region: 'us-west-2')
```

**New (Ruby 3.4.7) - Comment out or remove:**
```ruby
# ActionMailer::Base.add_delivery_method(:aws_sdk, Aws::SES::MessageDelivery, region: 'ap-northeast-1')
```

---

## Step 10: Update AL2023 Related Configuration

Edit `.ebextensions/00-expand-root-disk.config`:

```diff
---
- Resources:
-   AWSEBAutoScalingLaunchConfiguration:
-     Type: AWS::AutoScaling::LaunchConfiguration
-     Properties:
-       BlockDeviceMappings:
-         - DeviceName: /dev/xvda
-           Ebs:
-             VolumeSize:
-               32
+ option_settings:
+   aws:autoscaling:launchconfiguration:
+     RootVolumeSize: 32
```

Edit `.ebextensions/postgresql.config`:
```diff
- commands:
-   install_postgres11:
-     command: amazon-linux-extras install postgresql11 -y
+ packages:
+   yum:
+     postgresql15: []
+     libpq-devel: []

 files:
   "/etc/profile.d/z_psql.sh":
     content: |
       DATABASE_HOST=$(/opt/elasticbeanstalk/bin/get-config environment | jq -r 'with_entries(select(.key == "DATABASE_HOST")) | .[]')
       DATABASE_PASSWORD=$(/opt/elasticbeanstalk/bin/get-config environment | jq -r 'with_entries(select(.key == "DATABASE_PASSWORD")) | .[]')

       export PGHOST="${DATABASE_HOST}"
       export PGPORT=5432
       export PGDATABASE=safecast
       export PGUSER=safecast
       export PGOPTIONS=--search_path=public,postgis

       cat > ~/.pgpass <<EOF
       ${DATABASE_HOST}:5432:safecast:safecast:${DATABASE_PASSWORD}
       EOF
       chmod 600 ~/.pgpass
```

Rename `.ebextensions/cloudwatch.config` to `.ebextensions/cloudwatch.config`:
```diff
- packages:
-   yum:
-     awslogs: []
- 
- files:
-   "/etc/awslogs/awscli.conf" :
-     mode: "000600"
-     owner: root
-     group: root
-     content: |
-       [plugins]
-       cwlogs = cwlogs
-       [default]
-       region = `{"Ref":"AWS::Region"}`
- 
-   "/etc/awslogs/awslogs.conf" :
-     mode: "000600"
-     owner: root
-     group: root
-     content: |
-       [general]
-       state_file = /var/lib/awslogs/agent-state
- 
-   "/etc/awslogs/config/logs.conf" :
-     mode: "000600"
-     owner: root
-     group: root
-     content: |
-       [/var/log/messages]
-       log_group_name = `{"Fn::Join":["/", ["/aws/elasticbeanstalk", { "Ref":"AWSEBEnvironmentName" }, "var/app/current/log/production.log"]]}`
-       log_stream_name = {instance_id}
-       file = /var/app/current/log/production.log
- 
- commands:
-   "01":
-     command: systemctl enable awslogsd.service
-   "02":
-     command: systemctl restart awslogsd
+ option_settings:
+   aws:elasticbeanstalk:cloudwatch:logs:
+     StreamLogs: true
+     DeleteOnTerminate: false
+     RetentionInDays: 7
```

Edit `.ebextensions/tools.config`:
```diff
 packages:
   yum:
     htop: []
     ImageMagick: []
     sysstat: []
+    iftop: []
- 
- commands:
-   enable_epel:
-     command: amazon-linux-extras install epel -y
-   install_iftop:
-     command: yum -y install iftop
```

Edit `.ebextensions/web.config`:
```diff
- option_settings:
-   - namespace: aws:elb:policies
-     option_name: ConnectionSettingIdleTimeout
-     value: 600
- 
 commands:
   match_nginx_timeout_to_elb_timeout:
     command: |
       # If web tier (no sqs config) set nginx timeout to ELB timeout
       if ! /opt/elasticbeanstalk/bin/get-config meta -k sqsdconfig 2>/dev/null; then
         echo "proxy_read_timeout 600s;" > /etc/nginx/conf.d/web.conf
-        service nginx restart
+        systemctl restart nginx || true
       fi
```

Edit `.ebextensions/yarn.config`:
```diff
- packages:
-   rpm:
-     nodesource: https://rpm.nodesource.com/pub_10.x/el/7/x86_64/nodesource-release-el7-1.noarch.rpm
-   yum:
-     nodejs: []
- 
 commands:
-   install_yarn:
-     command: npm install --global yarn
+   01_install_node:
+     command: dnf install -y nodejs
```

Edit `config/application.rb`:
```diff
 ...
 module Safecast
   class Application < Rails::Application
     # Initialize configuration defaults for originally generated Rails version.
     config.load_defaults 5.2
+    config.active_support.cache_format_version = 7.1
     config.action_mailer.delivery_job = 'ActionMailer::MailDeliveryJob' # default 6.0
     config.active_record.belongs_to_required_by_default = false
     config.autoloader = :zeitwerk
...
```

Edit `config/database.yml`:
```diff
 default: &default
   username: safecast
   password:
   adapter: postgis
   encoding: unicode
   host: <%= ENV.fetch('DATABASE_HOST', 'localhost') %>
-  pool: 5
+  pool: 62
   port: 5432
   postgis_extension: postgis
   postgis_schema: postgis
   schema_search_path: public,postgis
   su_password:
   su_username: safecast
   timeout: 5000
+  variables:
+    statement_timeout: '40s'
```

Edit `app/models/ingest_measurement.rb`:
```diff
...
  class IngestMeasurement # rubocop:disable Metrics/ClassLength
   extend ActiveModel::Naming
   include Elasticsearch::Model

   index_name 'ingest-measurements-*'
-  document_type '_doc'

   class << self # rubocop:disable Metrics/ClassLength
     def data_for(query)
       search(query: query).results.map(&:_source)
     end
...
```

Create `config/initializers/multi_json_fix.rb`:
```diff
+ module MultiJson
+   def self.decode(string, options = {})
+     load(string, options)
+   end
+ end
```

compatibility clean-up
```bash
sed -i 's/cache_format_version = 6.1/cache_format_version = 7.0/' config/environments/development.rb
```

---

## Step 11: Run Tests

```bash
bundle exec rake test
# or
bundle exec rspec
```

---

## Step 12: Start the Application

```bash
bundle exec rails server -b 0.0.0.0 -p 3000
```

output logs
```text
$ bundle exec rails server -b 0.0.0.0 -p 3000
=> Booting Puma
=> Rails 7.1.6 application starting in development
=> Run `bin/rails server --help` for more startup options
DEPRECATION WARNING: Support for `config.active_support.cache_format_version = 6.1` has been deprecated and will be removed in Rails 7.2.

Check the Rails upgrade guide at https://guides.rubyonrails.org/upgrading_ruby_on_rails.html#new-activesupport-cache-serialization-format
for more information on how to upgrade.
 (called from <main> at /home/ec2-user/safecastapi/config/environment.rb:7)
Puma starting in single mode...
* Puma version: 7.1.0 ("Neon Witch")
* Ruby version: ruby 3.4.7 (2025-10-08 revision 7a5688e2a2) +PRISM [aarch64-linux]
*  Min threads: 5
*  Max threads: 5
*  Environment: development
*          PID: 202674
* Listening on http://0.0.0.0:3000
Use Ctrl-C to stop
```

For production:
```bash
RAILS_ENV=production bundle exec rails server -b 0.0.0.0 -p 3000
```

## One more thing

Log Rotation service for preventing storage full

```bash
sudo nano /etc/logrotate.d/safecastapi
```

Add this:
```bash
/home/ec2-user/safecastapi/log/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    copytruncate
    su ec2-user ec2-user
}
```

Explanation:
 - daily - Rotate logs daily
 - missingok - Don't error if log file is missing
 - rotate 14 - Keep 14 days of logs
 - compress - Compress old logs with gzip
 - delaycompress - Don't compress the most recent rotated log
 - notifempty - Don't rotate if log is empty
 - copytruncate - Copy log then truncate (keeps Rails writing to same file)
 - su ec2-user ec2-user - Run as ec2-user

Test it:
```bash
sudo logrotate -d /etc/logrotate.d/safecastapi
```

---

## Common Issues and Solutions

### Issue: `uninitialized constant ActiveSupport::LoggerThreadSafeLevel::Logger`
**Solution:** Upgrade to Rails 7.0+

### Issue: `cannot load such file -- mutex_m`
**Solution:** Add missing gems to Gemfile (see Step 5)

### Issue: `Could not be configured. It will not be installed. missing libffi`
**Solution:** Install system dependencies (see Step 1)

### Issue: Dependency conflicts with activerecord-postgis-adapter
**Solution:** Upgrade to version 9.0 (see Step 7)

### Issue: Server already running
**Solution:**
```bash
cat tmp/pids/server.pid | xargs kill
rm tmp/pids/server.pid
```

---

## Performance Monitoring

Check logs for performance metrics:
```bash
tail -f log/development.log | grep duration
```

Example output:
```json
{"method":"GET","path":"/","duration":245.0,"view":220.0,"db":22.0}
```

---

## Rollback Plan

If issues occur, rollback to Ruby 2.7:

```bash
rbenv global 2.7.8
cd ~/your-app
bundle install
bundle exec rails server -b 0.0.0.0 -p 3000
```

---

## Security Group Configuration

Ensure port 3000 is open in your EC2 security group:
- Type: Custom TCP
- Port: 3000
- Source: Your IP or 0.0.0.0/0 (for testing only)

---

## Production Deployment Checklist

- [ ] Backup database
- [ ] Test upgrade on staging environment
- [ ] Update environment variables
- [ ] Run database migrations
- [ ] Update systemd service (if applicable)
- [ ] Monitor logs for errors
- [ ] Run load tests
- [ ] Verify performance improvements

---

## Additional Resources

- [Ruby 3.4 Release Notes](https://www.ruby-lang.org/en/news/2024/12/25/ruby-3-4-0-released/)
- [Rails 7.1 Upgrade Guide](https://guides.rubyonrails.org/upgrading_ruby_on_rails.html)
- [rbenv Documentation](https://github.com/rbenv/rbenv)

---

## Support

For issues or questions, please open an issue in this repository.

---

## License

This guide is provided as-is for educational purposes.
