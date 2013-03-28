---
layout: post
title: "Automatically Generate Ember Models from Rails Serializers"
date: 2013-03-28 12:54
comments: true
categories:
---

*by [Matt Rogish](http://www.mattrogish.com/) ([@MattRogish](https://twitter.com/MattRogish))*

When working with [EmberJS](http://emberjs.com/) in a Rails context, we noticed that keeping
our Rails and Ember models in sync was a time-consuming and error-prone [process](http://techblog.fundinggates.com/blog/2012/10/moving-emberjs-forward/).

As [Jo](https://twitter.com/jo_liss) mentioned:

{% blockquote %}
In smaller projects you can repeat your database columns in the Ember-side
model definitions. For our more complex app, we found that this doesn't scale.
We ended up going with code generation for the ember-data model definitions,
generating a schema.js file using a rake task.
{% endblockquote %}

I've gotten some requests to expand on that and include some code. Note: this is
for pre-1.0 Ember so I wouldn't go copying-and-pasting this into your project.
It's not likely to work and you'll get strange errors. Still, since it's not much
code you could start with this and tweak it based on the latest version of Ember.


Consider the following Rails model, the proverbial "User":

{% codeblock app/models/user.rb lang:ruby %}

class User < ActiveRecord::Base
  attr_accessible :email, :first_name, :last_name, :phone, :birthdate

  belongs_to :organization

  def full_name
    "#{first_name} #{last_name}"
  end
end

{% endcodeblock %}

And the associated [ActiveModelSerializer](https://github.com/rails-api/active_model_serializers):

{% codeblock app/serializers/user_serializer.rb lang:ruby %}

class UserSerializer < ApplicationSerializer
  attributes :organization_id
  attributes :email, :first_name, :last_name, :phone

  has_one :organization

  # Computed attributes
  attributes :full_name, :ember_birthdate

  def ember_birthdate
    object.birthdate.strftime("%m/%d/%Y")
  end
end

{% endcodeblock %}

In order to consume this data, you'll need an Ember model that looks something like this (Coffeescript for brevity):

{% codeblock app/assets/javascripts/ember/models/user.js.coffee lang:coffeescript %}

App.User = DS.Model.extend
  email: DS.attr('string')
  first_name: DS.attr('string')
  last_name: DS.attr('string')
  phone: DS.attr('string')
  full_name: DS.attr('string')
  ember_birthdate: DS.attr('string')
  organization: DS.belongsTo('App.Organization')

{% endcodeblock %}

Kind of tedious to write all of that out, to remember to keep it up to date should you add, change, or delete something on the serializer side, and it just feels very un-DRY.
Why should we have to hand-edit multiple files when we make a change? That's why we have convention over configuration!

We created a simple rake task that uses the [schema](http://www.ruby-doc.org/gems/docs/a/active_model_serializers-0.5.2/ActiveModel/Serializer.html#method-c-schema)
capability of ActiveModelSerializers to convert it to JSON, then is processed by
the compiler to generate the model data.

Note: This may not work with the latest version of AMS.

The rake task is simple (you can shim it on rake db:migrate and elsewhere if you want):

{% codeblock lib/tasks/ember_schema.rake lang:ruby %}

namespace :db do
  namespace :schema do
    desc 'Regenerate the Ember schema.js based on the serializers'
    task :ember => :environment do
      schema_hash = {}
      Rails.application.eager_load! # populate descendants
      ApplicationSerializer.descendants.sort_by(&:name).each do |serializer_class|
        schema = serializer_class.schema
        schema_hash[serializer_class.model_class.name] = schema
      end

      schema_json = JSON.pretty_generate(schema_hash)
      File.open 'app/assets/javascripts/ember/models/schema.js', 'w' do |f|
        f << "// Model schema, auto-generated from serializers.\n"
        f << "// This file should be checked in like db/schema.rb.\n"
        f << "// Check lib/tasks/ember_schema.rake for documentation.\n"
        f << "window.serializerSchema = #{schema_json}\n"
      end
    end
  end
end

{% endcodeblock %}

Note that AMS does not include, nor care about, the Rails model validators, so
you'll need to handle that on your own. We wrote a small helper to output a
few basic validations but since Ember lacks built-in validators, you'd have
to write your own validator library.

[ember-validations](https://github.com/dockyard/ember-validations)
looks like a great library that supports all current (Rails3) validations.
You would just need to export the validations as JSON, and then write an appropriate converter.

So, great! We now have the JSON definition for the user:

{% codeblock app/assets/javascripts/ember/models/schema.js lang:javascript %}

// Model schema, auto-generated from serializers.
// This file should be checked in like db/schema.rb.
// Check lib/tasks/ember_schema.rake for documentation.
window.serializerSchema = {
  "User": {
    "attributes": {
      "id": "integer",
      "organization_id": "integer",
      "email": "string",
      "first_name": "string",
      "last_name": "string",
      "full_name": "string",
      "ember_birthdate": "string",
      "phone": "string"
    },
    "associations": {
      "organizations": {
        "belongs_to": "organization"
      }
    }
  }
}
{% endcodeblock %}

How do we get it into Ember?

{% codeblock app/assets/javascripts/ember/models/schema_parser.js.coffee lang:coffeescript %}

#= require frontend/models/schema

# Check lib/tasks/ember_schema.rake for documentation about the schema.

dsTypes =
  string: 'string'
  text: 'string'
  decimal: 'number'
  integer: 'number'
  boolean: 'boolean'
  date: 'date'
  # There is no time type in ember-data yet

# Define base classes like App.UserBase with attribute and
# association definitions based on schema data.
App.defineModelBaseClassesFromSchema = ->
  for className, schema of serializerSchema
    properties = {}

    for underscoredAttr, type of schema.attributes
      attr = underscoredAttr.camelize()
      if dsTypes[type]?
        if attr.match(/Id$/) and dsTypes[type] == 'number'
          # On the serializer side, we serialize belongs_to relationships as
          # integer _id fields, since AMS doesn't support belongs_to yet, and
          # has_one sideloads the association, causing infinite recursion.
          # Because of that, we infer a belongsTo relationship when we see _id
          # attributes in the schema.
          assoc = attr.replace(/Id$/, '')
          properties[assoc] = DS.belongsTo('App.' + assoc.capitalize())
        else
          properties[attr] = DS.attr(dsTypes[type])
      else
        # Ember.required doesn't quite do what we want it to yet, but maybe it
        # will be fixed. https://github.com/emberjs/ember.js/issues/1299
        properties[attr] = Ember.required()

    for assoc, info of schema.associations
      assoc = assoc.camelize()
      if tableName = info?.belongs_to
        properties[assoc] = DS.belongsTo('App.' + tableName.classify().capitalize())
      else if tableName = info?.has_many
        properties[assoc] = DS.hasMany('App.' + tableName.classify().capitalize())
      else if tableName = info?.has_one
        properties[assoc] = DS.belongsTo('App.' + tableName.classify().capitalize())

    # Do validator stuff here, if you so desire

    App["#{className}Base"] = App.Model.extend properties
{% endcodeblock %}

This will create the Ember model definition as above, except with a "Base" suffix (`UserBase`).
You can then extend it with Ember-only attributes:

{% codeblock app/assets/javascripts/ember/models/definitions.js.coffee lang:coffeescript %}
#= require frontend/models/schema_parser

App.Model = DS.Model.extend()

# Define base classes like App.UserBase based on the schema, which in
# turn is generated based on the serializers. Below, we only add server-side
# associations, because the schema has their types as
# `null`, as well as client-side computed properties.
#
# Check lib/tasks/ember_schema.rake for more documentation about the schema.
App.defineModelBaseClassesFromSchema()

App.User = App.UserBase.extend
  syncing: DS.attr('boolean')
  hasOrganizationBinding: 'organization.length'
{% endcodeblock %}

That's it. Now your models will be autogenerated and you
only have to worry about anything not included in the schema!