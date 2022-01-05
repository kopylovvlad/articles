# STI and multi attributes models in Rails

<!-- Photo by <a href="https://unsplash.com/@austindistel?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Austin Distel</a> on <a href="https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a> -->

![Photo by Philipp Berndt on Unsplash](image01.jpg)

Once I had a task to collect data from different landing pages to a database. The task is challenging because each web form from a landing page has different inputs and data. I figured out how to write an elegant solution in a Rails application.

## Preparation

First, we have to create a migration to store data in DB.

```ruby
class CreateLandingForms < ActiveRecord::Migration
  def change
    create_table :landing_forms do |t|
      t.string :type
      t.jsonb :data
      t.timestamps
    end
  end
end
```

The “type” is an important attribute for STI (single table inheritance) because we will have a few models (one model for each landing page) which use the same table in DB. Rails will  automatically keep class name to the attribute.

The “data” is an attribute with JSONB data type. Each model has a unique amount of fields. We will store the information in the attribute as JSON.

## Models

The main model looks like that. There is nothing special. Other models will be inherited from the main model.

```ruby
# app/models/landing_form.rb
# == Schema Information
#
# Table name: landing_forms
#
#  id         :bigint           not null, primary key
#  data       :jsonb
#  type       :string
#  created_at :datetime         not null
#  updated_at :datetime         not null
#
class LandingForm < ApplicationRecord
end
```

Let’s create models to collect data from different web forms. For instance, from 2 landing pages: Christmas’s landing page and Black Friday’s landing page.

Here is a model for Christmas’s landing page. Attributes are full name, phone number, city, state, address and a gift you would like to receive from Santa. :-) Method `store_accessor` helps us to collect data to the “data” field and work with it such typical ActiveRecord attributes.

```ruby
# app/models/landing_forms/christmas.rb
module LandingForms
  class Christmas < LandingForm
    store_accessor :data, :full_name, :phone_number, :city, :state, :address, :gift

    validates :full_name, presence: true
    validates :phone_number, presence: true
    validates :city, presence: true
    validates :state, presence: true
    validates :address, presence: true
    validates :gift, presence: true

    def self.permitted_params
      [:full_name, :phone_number, :city, :state, :address, :gift]
    end
  end
end
```

The next model is a model for Black Friday’s landing page. Attributes are first name, second name and email. If there is a requirement that email must be unique, we can add custom validation for email.

```ruby
# app/models/landing_forms/black_friday.rb
module LandingForms
  class BlackFriday < LandingForm
    store_accessor :data, :first_name, :last_name, :email

    scope :by_email, ->(email) { where(["data->>'email' IN (?)", email]) }

    validates :first_name, presence: true
    validates :last_name, presence: true
    validates :email, presence: true

    validate :unique_email

    def self.permitted_params
      [:first_name, :last_name, :email]
    end

    private

    def unique_email
      return if errors.size.positive?
      return if self.class.where(type: type).by_email(email).where.not(id: id).count.zero?

      errors.add(:email, I18n.t('landing_forms.email_already_taken'))
    end
  end
end
```

You see there is nothing difficult to describe behaviour. In order to prove that the validation works we will write test cases. Fortunately `store_accessor` works best with `FactoryBot`.

```ruby
# spec/factories/landing_form_factory.rb
FactoryBot.define do
  # factory for one model
  factory :black_friday_form, class: LandingForms::BlackFriday do
    first_name { Faker::Name.first_name }
    last_name { Faker::Name.last_name }
    email { Faker::Internet.email }
  end

  # factory for another model
  factory :christmas_form, class: LandingForms::Christmas do
    full_name { Faker::Name.name_with_middle }
    phone_number { Faker::PhoneNumber.phone_number }
    city { Faker::Address.city }
    state { Faker::Address.state }
    address { Faker::Address.full_address }
    gift { Faker::Hipster.sentence }
  end
end
```

And here are unit tests.

```ruby
# spec/models/landing_form_spec.rb
require 'rails_helper'

RSpec.describe LandingForms::Christmas, type: :model do
  let(:item) { build(:christmas_form) }

  it 'works' do
    expect(item.save).to eq(true)
  end
end

RSpec.describe LandingForms::BlackFriday, type: :model do
  let(:item) { build(:black_friday_form) }

  it 'works' do
    expect(item.save).to eq(true)
  end

  describe 'unique email validation' do
    let!(:item1) { create(:black_friday_form) }

    context 'email is not the same' do
      let(:item2) { build(:black_friday_form) }

      it 'is valid' do
        expect(item2.valid?).to eq(true)
        expect(item2.save).to eq(true)
      end
    end

    context 'email is the same' do
      let(:item2) { build(:black_friday_form, email: item1.email) }

      it 'is not valid' do
        expect(item2.valid?).to eq(false)
        expect(item2.errors.attribute_names).to include(:email)
        expect(item2.errors[:email]).to include(I18n.t('landing_forms.email_already_taken'))
      end
    end
  end
end
```

## Controllers

In order to receive data from a client, we write endpoints in rails-router, one endpoint for each controller.

```ruby
Rails.application.routes.draw do
namespace :api do
    namespace :v1 do
      namespace :landing_forms do
        post :black_friday, to: 'black_friday#create'
        post :christmas, to: 'christmas#create'
      end
    end
  end
end
```

A controller looks like that. Just one public method `create`. We don’t need anything more.

```ruby
# app/controllers/api/v1/landing_forms/black_friday_controller.rb
module Api
  module V1
    module LandingForms
      class BlackFridayController < ApplicationController
        def create
          object = model_class.new(model_params)
          if object.valid? && object.save
            render json: { success: true }
          else
            render json: object.errors, status: :unprocessable_entity
          end
        end

        private

        def model_params
          params.permit(*model_class.permitted_params)
        end

        def model_class
          ::LandingForms::BlackFriday
        end
      end
    end
  end
end
```

Let's write request test cases for the controller to prove that it works.

```ruby
# spec/requests/api/v1/landing_forms/black_friday_spec.rb
require 'rails_helper'

RSpec.describe Api::V1::SubscriptionsController, type: :request do
  describe '#create' do
    subject { post '/api/v1/landing_forms/black_friday', params: params, as: :json }

    before { subject }

    context 'email validation' do
      let!(:item1) { create(:black_friday_form) }

      context 'email is unique' do
        let(:params) do
          { first_name: Faker::Name.first_name, last_name: Faker::Name.last_name, email: Faker::Internet.email }
        end

        it 'renders success' do
          expect(response).to have_http_status(:ok)
          expect(JSON.parse(response.body)['success']).to eq(true)
        end
      end

      context 'email is the same' do
        let(:params) do
          { first_name: Faker::Name.first_name, last_name: Faker::Name.last_name, email: item1.email }
        end

        it 'returns errors' do
          expect(response).to have_http_status(:unprocessable_entity)
          json = JSON.parse(response.body)
          expect(json.keys).to eq(['email'])
          expect(json['email']).to include(I18n.t('landing_forms.email_already_taken'))
        end
      end
    end
  end
end
```

Voilà! It works. Now it’s super easy to collect data from one more web form. We need:
* Create a new model and describe all attributes and validations
* Create a new endpoint in the route and a controller


[dev.to](https://dev.to/kopylov_vlad/sti-and-multi-attributes-models-in-rails-12ce)
