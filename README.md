Writing

```sh
# ruby 3.0.4p208 (2022-04-12 revision 3fa771dded) [x86_64-linux]
rbenv install 3.0.4
rbenv global 3.0.4

gem install bundle
bundle install
bundle exec jekyll serve

watch_exec -c 'bundle exec jekyll serve' _config.yml -k
```
