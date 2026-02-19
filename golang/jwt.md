# JWT в golang

## Описание

Что такое JWT - token и для чего он нужен? Вообще JWT - это формат токена который позволяет передавать информацию об утверждении о пользователе в виде одной строки и проверять их подленность без обращении к базе данных. Сама идея JWT - сервер не хранит сессию и выдает клиенту подписанный документ. Клиент передает этот документ на сервер и сервер проверяет подлинность токена и выдает ответ. Если все ОК то клиент считается аутентефицированным.

## Как его используют и его синтаксис

JWT - состоит из трех частей через точку. Первая часть - `header`, вторая часть - `payload`, третья часть - `signature`.
`Header` и `payload` - если простыми словами это json формат. А вот `signature` - это подпись который проверяется выпушен ли он твоим сервером и чтобы `payload` после выдачи не менялся. Так же есть `secret_key` - он нужен чтобы проверить подлинность подписи.
Есть так же `утверждения` то есть `claims` - которые есть обязательные к заполнению и так же есть нащи собственные утвержения которые можно заполнить.

## Синтаксис

Опишем как вообще пишется это все и маленький пример с генерацией и проверкой JWT - токена.

```golang
  package jwt

  type Claims struct {
    ID int64 `json:"id"`
    jwt.RegisteredClaims
  }

  type Config struct {
    Secret []byte
    AccessTTL time.Duration
    Issuer string
  }

  type Service struct {
    secret []byte
    accessTTL time.Duration
    issuer string
  }

  func NewService(config Config) (*Service, error) {
    if len(config.Secret) == 0 {
      return nil, errors.New("secter is required")
    }
    if config.AccessTTL == 0 {
      config.AccessTTL = 15 * time.Minute
    }
    if config.Issuer == "" {
      return nil, errors.New("issuer is required")
    }

    return &Service{
      secret: config.Secret,
      accessTTL: config.AccessTTL,
      issuer: config.Issuer,
    }, nil
  }

  func (s *Service) GenerateToken(userID int64) (string, error) {
    if userID == 0 {
      return "", errros.New("user id is required")
    }

    now := time.Now()
    claims := Claims{
      ID: userID,
      RegisteredClaims: jwt.RegisteredClaims{
        Issuer: s.issuer,
        Subject: strconv.FormatInt(userID, 10),
        ExpiresAt: jwt.NewNumericDate(now.Add(s.accessTTL)),
        IssuedAt: jwt.NewNumericDate(now),
      },
    }

    token := jwt.NewWithClaims(jwt.SigningMethodHS256, claims)
    return token.SignedString(s.secret)
  }

  func (s *Service) ParseToken(tokenStr string) (*Claims, error) {
    tok, err := jwt.ParseWithClaims(tokenStr, &Claims{}, func(t *jwt.Token) (any, error) {
      if t.Method != jwt.SigningMethodHS256 {
        return nil, errors.New("unpexted signing method")
      }
      return s.secret, nil
    })
    if err != nil {
      return nil, err
    }

    claims, ok := tok.Claims.(*Claims)ы
    if !ok || !tok.Valid {
      return nil, errors.New("invalid token")
    }
    return claims, nil
  }
```

Тут в общем написан синтаксис написания JWT токена и предположительно какие функции мы описываем. Так же хотел бы сказать про вот эту часть в коде `jwt.SigningMethodHS256` - тут мы говорим каким методом мы подписываем, данный метод подписи чаще всего используется для одного сервиса. Можно использовать `jwt.SigningMethodHS512` если мы хотим что бы наши токены были достоверными и никому не удавались, а данный метод подписи чаще всего используется для нескольких сервисов.
