# Паттерны на ruby: Прокси

**Описание:**

Данный паттерн оборачивает собой другой объект (не меняя его интерфейс), притворяясь "оригинальный" объектом. Перехватывая все вызовы к нему, реализует дополнительную скрытую логику.
Прокси может использоваться для логирования запросов к объекту; для кеширования данных; для проверки доступа к объекту и прочих задач.

**Пример задачи:**

Представим что у нас есть рабочее rails приложение. В котором для пользователей есть функционал комментирования.
Необходимо добавить автоматическую блокировку комментариев содержащих спам.

**Реализация:**

Комментарии представлены моделью <Comment> в котором текст комментария находиться в атрибуте .message.
Функционал проверка на спам реализовать в модуле SpamFilter. У него есть метод .is_spam? который на вход принимает строку и возвращает true или false.
Сам прокси представлен в виде класса <CommentAntiSpamProxy> который оборачивает собой экземпляр класса <Comment>.
Прокси реализован таким образом что он запускает функционал проверки комментария на спам после того как был удачно вызван метод .save или .update_attributes.
Благодаря чему в контроллере CommentsController достаточно обернуть созданный экземпляр класса <Comment> в <CommentAntiSpamProxy> и дальше работать с ним как с обычным комментарием.

```ruby
module SpamFilter
  def self.is_spam?(msg)
    # magic code is here
  end
end

# == Schema Information
#
# Table name: comments
#
#  id             :integer          not null, primary key
#  message        :string
#  user_id        :integer
#  item_id        :integer
#  spam           :boolean
class Comment < ApplicationRecord
end


##
# proxy
class CommentAntiSpamProxy
  attr_accessor :real_comment
  def initialize(real_comment)
    @real_comment = real_comment
  end

  def method_missing(name, *args)
    answer = real_comment.send(name, *args)
    check_spam if need_to_check_spam?(name, answer)
    answer
  end

  private

  def need_to_check_spam?(name, answer)
    answer == true and (['save', 'update_attributes'].include?(name.to_s))
  end

  def check_spam
    return unless SpamFilter.is_spam?(real_comment.message)
    real_comment.update_attrubutes(spam: true)
  end
end

##
# controller
class CommentsController < ApplicationController
  def create
    @comment = CommentAntiSpamProxy.new(Comment.new(comment_params))
    if @comment.save
      render json: @comment, status: :ok
    else
      render json: @comment.errors, status: :unprocessable_entity
    end
  end

  def update
    @comment = CommentAntiSpamProxy.new(Comment.find(params[:comment_id]))
    if @comment.update_attributes(comment_params)
      render json: @comment, status: :ok
    else
      render json: @comment.errors, status: :unprocessable_entity
    end
  end

  private

  def comment_params
    params.require(:comment).permit(:user_id, :item_id, :message)
  end
end
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D0%BF%D1%80%D0%BE%D0%BA%D1%81%D0%B8-6dbc587e5703)
