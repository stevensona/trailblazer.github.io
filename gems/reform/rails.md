## Uniqueness Validation

Both ActiveRecord and Mongoid modules will support "native" uniqueness support from the model class when you use `validates_uniqueness_of`. They will provide options like `:scope`, etc.

You're encouraged to use Reform's non-writing `unique: true` validation, though. [Learn more](http://trailblazer.to/gems/reform/validation.html)

## ActiveModel Compliance

Forms in Reform can easily be made ActiveModel-compliant.

Note that this step is _not_ necessary in a Rails environment.

    class SongForm < Reform::Form
      include Reform::Form::ActiveModel
    end

If you're not happy with the `model_name` result, configure it manually via `::model`.

    class CoverSongForm < Reform::Form
      include Reform::Form::ActiveModel

      model :song
    end

`::model` will configure ActiveModel's naming logic. With `Composition`, this configures the main model of the form and should be called once.

This is especially helpful when your framework tries to render `cover_song_path` although you want to go with `song_path`.


## FormBuilder Support

To make your forms work with all the form gems like `simple_form` or Rails `form_for` you need to include another module.

Again, this step is implicit in Rails and you don't need to do it manually.

    class SongForm < Reform::Form
      include Reform::Form::ActiveModel
      include Reform::Form::ActiveModel::FormBuilderMethods
    end

### Simple Form

If you want full support for `simple_form` do as follows.

    class SongForm < Reform::Form
      include ActiveModel::ModelReflections

Including this module will add `#column_for_attribute` and other methods need by form builders to automatically guess the type of a property.


## Validation Shortform

Luckily, this can be shortened as follows.

    class SongForm < Reform::Form
      property :title, validates: {presence: true}
      property :length, validates: {numericality: true}
    end

Use `properties` to bulk-specify fields.

    class SongForm < Reform::Form
      properties :title, :length, validates: {presence: true} # both required!
      validates :length, numericality: true
    end



## Validations From Models

Sometimes when you still keep validations in your models (which you shouldn't) copying them to a form might not feel right. In that case, you can let Reform automatically copy them.

    class SongForm < Reform::Form
      property :title

      extend ActiveModel::ModelValidations
      copy_validations_from Song
    end

Note how `copy_validations_from` copies over the validations allowing you to stay DRY.

This also works with Composition.

    class SongForm < Reform::Form
      include Composition
      # ...

      extend ActiveModel::ModelValidations
      copy_validations_from song: Song, band: Band
    end


## ActiveRecord Compatibility

Reform provides the following `ActiveRecord` specific features. They're mixed in automatically in a Rails/AR setup.

 * Uniqueness validations. Use `validates_uniqueness_of` in your form.

As mentioned in the [Rails Integration](https://github.com/apotonick/reform#rails-integration) section some Rails 4 setups do not properly load.

You may want to include the module manually then.

    class SongForm < Reform::Form
      include Reform::Form::ActiveRecord

## Mongoid Compatibility

Reform provides the following `Mongoid` specific features. They're mixed in automatically in a Rails/Mongoid setup.

 * Uniqueness validations. Use `validates_uniqueness_of` in your form.

You may want to include the module manually then.

    class SongForm < Reform::Form
      include Reform::Form::Mongoid



## Troubleshooting

1. In case you explicitely _don't_ want to have automatic support for `ActiveRecord` or `Mongoid` and form builder: `require reform/form`, only.
2. In some setups around Rails 4 the `Form::ActiveRecord` module is not loaded properly, usually triggering a `NoMethodError` saying `undefined method 'model'`. If that happened to you, `require 'reform/rails'` manually at the bottom of your `config/application.rb`.
3. Mongoid form gets loaded with the gem if `Mongoid` constant is defined.