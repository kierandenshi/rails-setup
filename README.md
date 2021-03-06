# rails-setup
Setup notes for new Rails apps.

## Use UUID indexes

Create a migration like this:
```
class EnablePgcryptoExtension < ActiveRecord::Migration[6.1]
  def change
    enable_extension 'pgcrypto'
  end
end
```

### ActiveStorage
The migration created by `rails active_storage:install` will need to be edited to use UUID. 

*Remember to add `type: :uuid` to all `belongs_to` and `references` definitions.
```
# This migration comes from active_storage (originally 20170806125915)
class CreateActiveStorageTables < ActiveRecord::Migration[5.2]
  def change
    create_table :active_storage_blobs, id: :uuid do |t|
      t.string   :key,          null: false
      t.string   :filename,     null: false
      t.string   :content_type
      t.text     :metadata
      t.string   :service_name, null: false
      t.bigint   :byte_size,    null: false
      t.string   :checksum,     null: false
      t.datetime :created_at,   null: false

      t.index [ :key ], unique: true
    end

    create_table :active_storage_attachments, id: :uuid do |t|
      t.string     :name,     null: false
      t.references :record,   null: false, polymorphic: true, index: false
      t.references :blob,     null: false, type: :uuid

      t.datetime :created_at, null: false

      t.index [ :record_type, :record_id, :name, :blob_id ], name: "index_active_storage_attachments_uniqueness", unique: true
      t.foreign_key :active_storage_blobs, column: :blob_id
    end

    create_table :active_storage_variant_records, id: :uuid do |t|
      t.belongs_to :blob, null: false, index: false, type: :uuid
      t.string :variation_digest, null: false

      t.index %i[ blob_id variation_digest ], name: "index_active_storage_variant_records_uniqueness", unique: true
      t.foreign_key :active_storage_blobs, column: :blob_id
    end
  end
end
```

## Old migrations
Delete old migrations then add a new migration with the same TIMESTAMP as the last migration you deleted, with the following content:
```
class OldMigrations < ActiveRecord::Migration[5.1]
  REQUIRED_VERSION = TIMESTAMP
  def up
    if ActiveRecord::Migrator.current_version < REQUIRED_VERSION
      raise StandardError, "`rails db:schema:load` must be run prior to `rails db:migrate`"
    end
  end
end
```

## Sidekiq
Clear all queued jobs in the Rails console with `Sidekiq.redis(&:flushdb)` (best not do this in production)
