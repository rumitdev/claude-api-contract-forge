# Ruby on Rails — Build Templates

Use these templates when the project detection (Step 0.1) identifies Rails. Adapt all placeholder tokens to the actual resource name. Use snake_case per Ruby conventions.

Generate: Controller, Model, Serializer, Migration, Route, Request validation. Generate only the operations the user requested.

---

## File 1: Model — `app/models/{resource}.rb`

```ruby
class {Resource} < ApplicationRecord
  # Validations
  validates :title, presence: true, length: { maximum: 200 }
  validates :amount, presence: true, numericality: { greater_than_or_equal_to: 0 }
  validates :status, inclusion: { in: %w[draft sent paid] }, allow_nil: true

  # Defaults
  attribute :status, :string, default: "draft"

  # Scopes
  scope :search, ->(query) { where("title ILIKE ?", "%#{query}%") if query.present? }
  scope :by_status, ->(status) { where(status: status) if status.present? }
  scope :sorted, ->(sort_by, order) {
    order(sort_by => order || :desc)
  }
end
```

---

## File 2: Serializer — `app/serializers/{resource}_serializer.rb`

Using `active_model_serializers` or `jsonapi-serializer` (detect from Gemfile):

```ruby
# With active_model_serializers
class {Resource}Serializer < ActiveModel::Serializer
  attributes :id, :title, :amount, :status, :created_at, :updated_at

  def created_at
    object.created_at.iso8601
  end

  def updated_at
    object.updated_at.iso8601
  end
end
```

If the project uses `jbuilder` instead:

```ruby
# app/views/{resources}/show.json.jbuilder
json.extract! @{resource}, :id, :title, :amount, :status
json.created_at @{resource}.created_at.iso8601
json.updated_at @{resource}.updated_at.iso8601
```

---

## File 3: Controller — `app/controllers/api/v1/{resources}_controller.rb`

```ruby
module Api
  module V1
    class {Resources}Controller < ApplicationController
      before_action :authenticate_user!
      before_action :set_{resource}, only: [:show, :update, :destroy]

      # GET /api/v1/{resources}
      def index
        {resources} = {Resource}.all
        {resources} = {resources}.search(params[:search])
        {resources} = {resources}.by_status(params[:status])
        {resources} = {resources}.sorted(
          params[:sort_by] || "created_at",
          params[:sort_order]
        )

        page = (params[:page] || 1).to_i
        limit = [[params[:limit]&.to_i || 10, 100].min, 1].max
        total = {resources}.count
        {resources} = {resources}.offset((page - 1) * limit).limit(limit)

        render json: {
          items: ActiveModelSerializers::SerializableResource.new({resources}),
          total: total,
          page: page,
          limit: limit,
          totalPages: (total.to_f / limit).ceil
        }
      end

      # POST /api/v1/{resources}
      def create
        {resource} = {Resource}.new({resource}_params)

        if {resource}.save
          render json: {resource}, serializer: {Resource}Serializer, status: :created
        else
          render json: { errors: {resource}.errors.full_messages }, status: :unprocessable_entity
        end
      end

      # GET /api/v1/{resources}/:id
      def show
        render json: @{resource}, serializer: {Resource}Serializer
      end

      # PUT /api/v1/{resources}/:id
      def update
        if @{resource}.update({resource}_params)
          render json: @{resource}, serializer: {Resource}Serializer
        else
          render json: { errors: @{resource}.errors.full_messages }, status: :unprocessable_entity
        end
      end

      # DELETE /api/v1/{resources}/:id
      def destroy
        @{resource}.destroy
        head :no_content
      end

      private

      def set_{resource}
        @{resource} = {Resource}.find(params[:id])
      end

      def {resource}_params
        params.require(:{resource}).permit(:title, :amount, :status)
      end
    end
  end
end
```

---

## File 4: Migration

```ruby
class Create{Resources} < ActiveRecord::Migration[7.1]
  def change
    create_table :{resources}, id: :uuid do |t|
      t.string :title, null: false, limit: 200
      t.decimal :amount, null: false, precision: 12, scale: 2
      t.string :status, default: "draft"
      t.timestamps
    end

    add_index :{resources}, :status
    add_index :{resources}, :created_at
  end
end
```

Run: `rails db:migrate`

---

## File 5: Routes — `config/routes.rb`

```ruby
Rails.application.routes.draw do
  namespace :api do
    namespace :v1 do
      resources :{resources}, only: [:index, :show, :create, :update, :destroy]
    end
  end
end
```

If the user requested only specific operations, use the `only` option to limit which routes are generated.
