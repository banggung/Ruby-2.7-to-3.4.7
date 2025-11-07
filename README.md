# Ruby 2.7 to 3.4.7 Upgrade Guide

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

## Step 1: Install System Dependencies

```bash
sudo dnf install -y gcc make openssl-devel zlib-devel readline-devel \
  libffi-devel libyaml-devel bzip2
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
rbenv install 3.4.7
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
ruby '3.4.7'
```

---

## Step 5: Add Missing Standard Library Gems

Ruby 3.4 removed several gems from the default bundle. Add these to your Gemfile:

```ruby
gem 'mutex_m'
gem 'ostruct'
gem 'bigdecimal'
gem 'drb'
gem 'logger'
gem 'csv'
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

```bash
gem install bundler
bundle install
```

If you encounter dependency conflicts:
```bash
bundle update
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

## Step 10: Update Database Configuration

Edit `config/database.yml`:

```yaml
production:
  adapter: postgresql
  encoding: unicode
  database: your_database
  username: your_username
  password: <%= ENV['DATABASE_PASSWORD'] %>
  host: your-rds-endpoint.rds.amazonaws.com
  port: 5432
  pool: 5
```

Set environment variable:
```bash
export DATABASE_PASSWORD='your_password'
echo "export DATABASE_PASSWORD='your_password'" >> ~/.bashrc
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

For production:
```bash
RAILS_ENV=production bundle exec rails server -b 0.0.0.0 -p 3000
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
