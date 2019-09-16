---
title: Rails 发现 - Validator 实例变量的坑
date: 2018-09-10 22:05:47
tags: [ruby, rails]
categories: [Web Backend]
---

当我们想复用或分离 ActiveModel 里的 Validation 模块时，会使用 [Custom Validator](https://guides.rubyonrails.org/active_record_validations.html#custom-validators)。在某些情况下，Validator 保存实例变量会产生预期之外的事情。

<!--more-->

## 问题

下面是一个校验 Webhook 的 url 是否含私有 IP 地址或禁用域名的 Validator

```ruby
# url_validator.rb

require 'ipaddr'

class UrlValidator < ActiveModel::EachValidator

  PRIVATE_IPS = [
    IPAddr.new('10.0.0.0/8'),
    IPAddr.new('172.16.0.0/12'),
    IPAddr.new('192.168.0.0/16')
  ].freeze

  FORBIDDEN_HOSTS = %w[gitee.com localhost].freeze

  def validate_each(record, attribute, value)
    if private_ip?(value)
      record.errors.add(attribute, :is_private_ip)
    elsif forbidden_host?(value)
      record.errors.add(attribute, :is_forbidden_host)
    end
  end

  private

  def get_host(value)
    @host ||= URI(value).host rescue nil
  end

  def private_ip?(url)
    ip_address = IPAddr.new(get_host(url)) rescue nil
    PRIVATE_IPS.any? {|private_ip| private_ip.include?(ip_address)} if ip_address
  end

  def forbidden_host?(url)
    host = get_host(url)
    FORBIDDEN_HOSTS.any? {|forbidden_host| forbidden_host == host.downcase} if host
  end
end
```

```ruby
# web_hook.rb

class WebHook < ActiveRecord::Base
  validates url, url: true
end
```

UrlValidator 在调用 get_host 方法时，将 url 的 host 保存在 @host 中。

看似很正常的操作，在 Production 环境下出问题了！！ 当在校验一个非合法的 url 后，再校验一个合法的 url，其会出现验证失败。

## 分析

我们从 ActiveModel 的 validates 方法入手

```ruby
# rails-3-2/activemodel/lib/active_model/validations
module ActiveModel
  module Validations
    module ClassMethods
      def validates(*attributes)
        defaults = attributes.extract_options!
        validations = defaults.slice!(*_validates_default_keys)

        raise ArgumentError, "You need to supply at least one attribute" if attributes.empty?
        raise ArgumentError, "You need to supply at least one validation" if validations.empty?

        defaults.merge!(:attributes => attributes)

        validations.each do |key, options|
          key = "#{key.to_s.camelize}Validator"

          begin
            validator = key.include?('::') ? key.constantize : const_get(key)
          rescue NameError
            raise ArgumentError, "Unknown validator: '#{key}'"
          end

          validates_with(validator, defaults.merge(_parse_validates_options(options)))
        end
      end

      def validates_with(*args, &block)
        options = args.extract_options!
        args.each do |klass|
          validator = klass.new(options, &block)     # 注意这里
          validator.setup(self) if validator.respond_to?(:setup)

          if validator.respond_to?(:attributes) && !validator.attributes.empty?
            validator.attributes.each do |attribute|
              _validators[attribute.to_sym] << validator
            end
          else
            _validators[nil] << validator
          end

          validate(validator, options)
        end
      end
    end
  end
end
```

发现 Rails 在加载一个 WebHook 类时，会生成一个 UrlValidator 对象，在每个 WebHook 对象校验时都调用上面 UrlValidator 对象进行校验。

而在 Production 环境下默认开启了 cache_classes，其不会在每次请求时重新加载 WebHook 类。这就出现问题了， @host 在第一次请求被保存起来了, 导致之后请求出现问题。

## 结论

从这个例子我们可以总结出，Validator 不应该保存校验对象的相关数据！