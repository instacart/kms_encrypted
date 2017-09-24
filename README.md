# KMS Encrypted

[KMS](https://aws.amazon.com/kms/) + [attr_encrypted](https://github.com/attr-encrypted/attr_encrypted)

The attr_encrypted gem is great for encryption, but:

1. Leaves you to manage the security of your keys
2. Doesn’t provide a great audit trail to see how data has been accessed

KMS addresses both of the issues and it’s easy to use them together.

**Note:** This has not been battle-tested in a production environment, so use with caution

## Getting Started

Add this line to your application’s Gemfile:

```ruby
gem 'kms_encrypted'
```

Add a column to store encrypted KMS data keys

```ruby
add_column :users, :encrypted_kms_key, :string
```

Create a [KMS master key](https://console.aws.amazon.com/iam/home#/encryptionKeys) and set it in your environment (we recommend [dotenv](https://github.com/bkeepers/dotenv))

```sh
KMS_KEY_ID=arn:aws:kms:...
```

And update your model

```ruby
class User < ApplicationRecord
  has_kms_key ENV["KMS_KEY_ID"]

  attr_encrypted :email, key: :kms_key
end
```

For each encrypted attribute, use the `kms_key` method for its key.

## Auditing

[AWS CloudTrail](https://aws.amazon.com/cloudtrail/) logs all decryption calls. However, to know what data is being decrypted, you’ll need to add context.

Add a `kms_encryption_context` method to your model.

```ruby
class User < ApplicationRecord
  def kms_encryption_context
    self.id ||= self.class.connection.execute("select nextval('#{self.class.sequence_name}')").first["nextval"]
    {"Record" => "#{model_name}/#{id}"}
  end
end
```

The context is used as part of the encryption and decryption process, so it must be a value that doesn’t change. Otherwise, you won’t be able to decrypt.

Read more about [encryption context here](http://docs.aws.amazon.com/kms/latest/developerguide/encryption-context.html).

## How It Works

KMS allows you to generate data keys from your master key. Data keys are encrypted and stored with each record. Whenever you want to decrypt data, you need to first decrypt the data key with KMS. Once you have the decrypted key, you can then use it to decrypt the data with attr_encrypted.

## TODO

- add support for multiple data keys per record

## History

View the [changelog](https://github.com/ankane/kms_encrypted/blob/master/CHANGELOG.md)

## Contributing

Everyone is encouraged to help improve this project. Here are a few ways you can help:

- [Report bugs](https://github.com/ankane/kms_encrypted/issues)
- Fix bugs and [submit pull requests](https://github.com/ankane/kms_encrypted/pulls)
- Write, clarify, or fix documentation
- Suggest or add new features