# Middleware in Golang

## Описание

Тема middleware — одна из важных тем при разработке backend-сервиса. Что вообще такое middleware?

Middleware — это функция-обёртка вокруг обработчика. В чём заключается идея: есть `следующий` обработчик `next`, а middleware делает что-то до него или вообще не пропускает запрос дальше.

Важно: если middleware не вызывает `next.ServeHTTP(w, r)`, то запрос дальше не пойдёт — цепочка обработки остановится.

## Пример

Разберём несколько примеров:

```go
func myMW(next http.Handler) http.Handler {
  return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    // тут идёт код ДО следующего обработчика
    next.ServeHTTP(w, r)
    // тут идёт код ПОСЛЕ
  })
}
```

Сам по себе middleware — это паттерн, а пример который мы разбираем — это самый частый вариант middleware, который проверяет авторизован ли человек, какая у него роль и т.д. Чаще всего это и проверяется: если пользователь авторизован и его роль совпадает, то он имеет доступ к определённым endpoint’ам. Также естественно есть публичные endpoint’ы, которые не требуют авторизации.

Ещё вот такой пример middleware: тут уже будет происходить аутентификация (кто пользователь) и авторизация (что ему можно), но мы не будем его разбирать.

```go
func Auth(jwt *token.Service) middleware {
  return func(next http.Handler) http.Handler {
    return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
      h := r.Header.Get("Authorization")
      const prefix = "Bearer "
      if !strings.HasPrefix(h, prefix) {
        http.Error(w, "missing token", http.StatusUnauthorized)
        return
      }

      raw := strings.TrimSpace(strings.TrimPrefix(h, prefix))
      claims, err := jwt.ParseToken(raw)
      if err != nil {
        http.Error(w, err.Error(), http.StatusUnauthorized)
        return
      }

      ctx := context.WithValue(r.Context(), ctxUserID, claims.ID)
      ctx = context.WithValue(ctx, ctxRole, claims.Role)

      // передаём данные дальше через контекст
      next.ServeHTTP(w, r.WithContext(ctx))
    })
  }
}
```

Тут у нас идёт проверка на авторизацию: проверяется, есть ли у пользователя `jwt токен`, и если есть — мы его парсим и кладём данные в `context`. Также мы пробрасываем через `context` роль пользователя, чтобы следующие middleware могли это проверить.
