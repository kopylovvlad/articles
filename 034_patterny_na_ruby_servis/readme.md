# Паттерны на ruby: Сервис

**Описание:** Данный паттерн навязывает структуру писать отдельный объект для описания каждой бизнес-логики. В идеале, этот объект имеет только один публичный метод в котором последовательно вызываются все приватные.

**Пример задачи:** Необходимо реализовать функционал запроса на восстановление пароля.

**Реализация:** Создадим класс <ForgottenPasswordService> для описание следующего алгоритма действий: Найти пользователя по его email; Сгенерировать токен восстановления пароля; Отправить сообщение со ссылкой-восстановления на почту.

```ruby
class ForgottenPasswordService
  def initialize(user_email)
    @user_email = user_email
    @user = nil
  end

  def perform
    find_user && generate_token && send_email
  end

  private

    def find_user
      @user = User.find_by(email: @user_email)
      @user.present?
    end

    def generate_token
      @user.update_attributes(token: TokenGenerator.generate)
    end

    def send_email
      UserMailer.password_reset(@user.email, @user.token)
    end
end
```

[Medium](https://kopilov-vlad.medium.com/%D0%BF%D0%B0%D1%82%D1%82%D0%B5%D1%80%D0%BD%D1%8B-%D0%BD%D0%B0-ruby-%D1%81%D0%B5%D1%80%D0%B2%D0%B8%D1%81-5940fa2502a9)
