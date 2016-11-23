#kms_attrs

kms_attrs is a gem for easily adding Amazon Web Services KMS encryption to your ActiveRecord model attributes. It uses the GenerateDataKey method to perform "envelope" encryption locally with an OpenSSL AES-256-CBC cipher.

To use, simply put the following code in your models for the fields you want to encrypt:
```ruby
kms_attr :my_attribute, key_id: 'my-aws-kms-key-id'
```
Encryption is done at time of assignment.

To retrieve the decrypted data, call:
```ruby
  my_model_instance.my_attribute_d
```

Encrypted data is stored as a [MessagePack](https://github.com/msgpack/msgpack-ruby) blob in your database in the `#{my_attribute}_enc` column. It should be a binary column of sufficient size to store the encrypted data + metadata (suggested 65535).

##Additional Options
You can add encryption contexts as strings, method calls, or procs. Default is none.
```ruby
kms_attr :my_attribute, key_id: 'my-aws-kms-key-id',
  context_key: 'my context key', context_value: 'my context value'

kms_attr :my_attribute, key_id: 'my-aws-kms-key-id',
  context_key: :model_method_context_key, context_value: :model_method_context_value

kms_attr :my_attribute, key_id: 'my-aws-kms-key-id',
  context_key: Proc.new { }, context_value: Proc.new { }
```

You can also toggle whether or not the model instance should retain decrypted values. Default is false. Change to true if you want to reduce the AWS API calls made for constant decryption. I cannot comment on the security implications enabling or disabling retaining.
```ruby
kms_attr :my_attribute, key_id: 'my-aws-kms-key-id',
  retain: true
```

To clear a retained decrypted value, call:
```ruby
  my_model_instance.my_attribute_clear
```

This will attempt mutate the stored string to contain just null bytes, and then dereference it to be garbage collected. No guarantees are provided about additional copies of the retained data being cached elsewhere.

##Aws Configuration
This gem expects some standard Aws SDK configuration and some not so standard. The Aws client is initiated with no credentials. This should then load credentials either from ENV['AWS_ACCESS_KEY_ID'] and ENV['AWS_SECRET_ACCESS_KEY'] or an IAM role on an EC2 instance.

You can configure your region in a Rails initializer with;
```ruby
Aws.config[:region] = 'us-east-1'
```

or by using the documented AWS environmental variables.


###Notes
This gem has been developed against Ruby 2.1.5, Rails 4.2, and AWS SDK v2. Credit where credit is due, I used strongbox by spikex as an inspiration and guide when creating this. https://github.com/spikex/strongbox

###Disclaimer
I make no claims about enhanced security when using this gem.

###To Do
* Tests
* Choose your own encryption method
* Choose your own KMS key type
* Specify AWS region in configuration

###Read more about AWS KMS
* http://aws.amazon.com/kms/
* http://docs.aws.amazon.com/sdkforruby/api/Aws/KMS/Client.html
