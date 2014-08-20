---
layout: post
title: "Nested form with Rails 4.1"
date: 2014-08-18 16:50:01 +0800
comments: true
categories: [RailsCasts, Rails, Tips]
---
Recently I got a [guide](http://us5.campaign-archive2.com/?u=0d868b8bb90703d75a27d8b42&id=055f149177&e=eea34cb3a8) of telling us how to be self-taught from on-line courses. It shows me how important taking note is. I highly recommended this guide to who also wants to be self-taught.

> The average person forgets 40% of what they learn within 20 minutes.

So I decide to take notes here since now. It will spend most of my time to do, but if I do not do it today, I know I won't do it anymore. That is my lazy personality. The first technical post I am going to present my reviews of [#196 Nested Model Form (revised)](http://railscasts.com/episodes/196-nested-model-form-revised); meanwhile, some of the codes there are not working because of the version of Rails, so I will try to make it work like video shows with the latest Rails, please revise my code if you have better ways, thanks a bunch. OK so let us get started.

Like [#196 Nested Model Form (revised)](http://railscasts.com/episodes/196-nested-model-form-revised) said, there are three separate models and already had associations with each other:

``` ruby app/models/survey.rb
class Survey < ActiveRecord::Base
  has_many :questions
end
```

``` ruby app/models/question.rb
class Question < ActiveRecord::Base
  belongs_to :survey
  has_many :answers
end
```

``` ruby app/models/answer.rb
class Answer < ActiveRecord::Base
  belongs_to :question
end
```

Here is the schema:

``` ruby db/schema.rb
ActiveRecord::Schema.define(version: 20140817155213) do

  create_table "answers", force: true do |t|
    t.text     "content"
    t.integer  "question_id"
    t.datetime "created_at"
    t.datetime "updated_at"
  end

  create_table "questions", force: true do |t|
    t.text     "content"
    t.integer  "survey_id"
    t.datetime "created_at"
    t.datetime "updated_at"
  end

  create_table "surveys", force: true do |t|
    t.string   "name"
    t.datetime "created_at"
    t.datetime "updated_at"
  end

end
```
## The Purpose

All we want is to manage them in a single form. In order to implement this, we have to call `accepts_nested_attributes_for` method in our model:

``` ruby app/models/survey.rb
class Survey < ActiveRecord::Base
  ...
  accepts_nested_attributes_for :questions
end
```

So that we can add `fields_for` to `Question` model in the form:

``` html app/views/surveys/_form.html.erb
...
...
...
<div class="field">
  <%= f.label :name %><br>
  <%= f.text_field :name %>
</div>

<%= f.fields_for :questions do |builder| %>
  <fieldset>
  <%= builder.label :content, 'Question' %><br>
  <%= builder.text_area :content %><br>
  </fieldset>
<% end %>
...
...
...

```

Remember to add plural name of associated model (Followed by `_attributes`) to `Strong Parameter` from `SurveysController`:

``` ruby app/controllers/surveys_controller.rb
...
...
...
private
  ...
  ...
  ...

  # Never trust parameters from the scary internet, only allow the white list through.
  def survey_params
    params.require(:survey).permit(:name, questions_attributes: [:id, :content])
  end
```

## Removing Questions

Next we are going to implement removing questions feature. We will create a checkbox which its key is `_destroy`, so the question will be removed if checkbox is checked.

``` html app/views/surveys/_form.html.erb
...
...
...
<div class="field">
  <%= f.label :name %><br>
  <%= f.text_field :name %>
</div>

<%= f.fields_for :questions do |question| %>
  <fieldset>
  <%= question.label :content, 'Question' %><br>
  <%= question.text_area :content %><br>
  <%= question.check_box :_destroy %>
  <%= question.label :_destroy, 'Remove question' %>
  </fieldset>
<% end %>
```
Add `allow_destroy` option to `Survey` model, set the value to be true:

``` ruby app/models/survey.rb
class Survey < ActiveRecord::Base
  ...
  accepts_nested_attributes_for :questions, allow_destroy: true
end
```

We also need to add `_destroy` to `questions_attributes` (Strong Parameter)

```ruby app/controllers/surveys_controller.rb
...
...
...
private
  ...
  ...
  ...

  # Never trust parameters from the scary internet, only allow the white list through.
  def survey_params
    params.require(:survey).permit(:name, questions_attributes: [:id, :content, :_destroy])
  end
```

Finally we can delete the questions through checking the checkboxes.

## Editing Answers

Now, we also want to edit each question's answers.

Just like before, we can call `accepts_nested_attributes_for` method to its associated model -  `Question`:

``` ruby app/models/question.rb
class Question < ActiveRecord::Base
  ...
  accepts_nested_attributes_for :answers, allow_destroy: true
end
```

Remember again, insert the plural name of associated model, `Question` model in the strong parameter. Let's see how to do it:

``` ruby app/controllers/surveys_controller.rb
...
...
...
private
  ...
  ...
  ...

  # Never trust parameters from the scary internet, only allow the white list through.
  def survey_params
    params ... questions_attributes: [:id, :content, :_destroy, answers_attributes: [:id, :content, :_destroy]])
  end
```

Next, add fields for `Answer`:

``` html app/views/surveys/_form.html.erb
...
...
<%= f.fields_for :questions do |question| %>
  <fieldset>
  <%= question.label :content, 'Question' %><br>
  <%= question.text_area :content %><br>
  <%= question.check_box :_destroy %>
  <%= question.label :_destroy, 'Remove question' %>
  <%= question.fields_for :answers do |answer| %>
    <%= answer.label :content, 'Answer' %><br>
    <%= answer.text_field :content %><br>
  <% end %>
  </fieldset>
<% end %>
...
...
```

Then `Answer` model can be managed by the form.

But it it better that we can make our form more neat, we will use `render` method to do it.

``` html app/views/surveys/_form.html.erb
...
...
<%= f.fields_for :questions do |question| %>
  <%= render 'question_fields', f: question %>
<% end %>
...
...
```

And create `_question_fields.html.erb` file into `app/views/surveys` folder:

```html app/views/surveys/_question_fields.html.erb
<fieldset>
  <%= f.label :content, 'Question' %><br>
  <%= f.text_area :content %><br>
  <%= f.check_box :_destroy %>
  <%= f.label :_destroy, 'Remove question' %>
  <%= f.fields_for :answers do |answer| %>
    <%= answer.label :content, 'Answer' %><br>
    <%= answer.text_field :content %><br>
  <% end %>
</fieldset>
```

`render` each question's answers:

```html app/views/surveys/_question_fields.html.erb
<fieldset>
  <%= f.label :content, 'Question' %><br>
  <%= f.text_area :content %><br>
  <%= f.check_box :_destroy %>
  <%= f.label :_destroy, 'Remove question' %>
  <%= f.fields_for :answers do |answer| %>
    <%= render 'answer_fields', f: answer %>
  <% end %>
</fieldset>
```

create `_answer_fields.html.erb` into `app/views/surveys` folder:

``` html app/views/surveys/_answer_fields.html.erb
<fieldset>
  <%= f.label :content, 'Answer' %>
  <%= f.text_field :content %>
  <%= f.check_box :_destroy %>
  <%= label :_destroy, 'Remove' %>
</fieldset>
```

## TODO: Editing Questions and Answers through JavaScript