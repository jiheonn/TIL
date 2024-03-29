# Passport

Node.js 를 위한 인증 미들웨어이며, Express 기반 웹에서 유용하게 사용할 수 있음  
Facebook, Twitter 등 소셜 로그인 인증을 지원

> Passport is authentication middleware for Node.js. Extremely flexible and modular, Passport can be unobtrusively dropped in to any Express-based web application. A comprehensive set of strategies support authentication using a username and password, Facebook, Twitter, and more.

## Passport 설치

- `passport-local` : 직접 구현할 때 사용
- `express-session` : passport를 통해 로그인 후 유저 정보를 세션에 저장하기 위해 사용

```bash
$ npm install passport passport-local express-session
```

> 소셜 로그인 인증을 위한 패키지
>
> ```bash
> passport-google-oauth
> passport-facebook
> passport-twitter
> passport-kakao
> passport-naver
> ```

## Session 및 Passport 설정

`auth/` 내에 `passport.js`와 `session.js`를 생성합니다.

그리고 프로젝트 루트 경로 내에 `.env`를 생성한 후 다음과 같이 작성합니다.

`SESSION_SECRET`은 임의로 작성한 것이므로, 원하는 값으로 변경해도 좋습니다.

해당 정보는 인증에 관련되어 있어 배포 과정에서 노출되면  
보안적인 이슈가 발생할 수 있기 때문에  
이와 같이 환경 변수 파일을 만들어 관리합니다.

```
// .env

SESSION_SECRET=d4SDLFKD24SDfd3s
```

```
|-- src
     |-- auth
     |   `-- passport.js
     |   `-- session.js
     |-- app.js
|-- .env
```

### Session 설정

`ES6` 문법 기준으로 `auth/session.js` 파일을 작성합니다.

`dotenv`는 앞에서 작성한 환경 변수 파일인 `.env`에 접근하기 위해 사용되는 패키지 입니다.
만약, 존재하지 않다면 다음 명령어로 설치할 수 있습니다.

```bash
$ npm install dotenv
```

> 패키지를 `import` 한 후 `dotenv.config()` 함수를 호출하면  
> `process.env.변수명`으로 접근이 가능합니다.

**express-session option**

- `secret` : 필수 옵션이며, 세션을 암호화할 때 사용
- `cookie`
  - `path` : 쿠키 경로 설정
  - `httpOnly` : 클라이언트 측 자바스크립트를 통하여 쿠키에 접근을 제한
  - `secure` : HTTPS 필요
  - `maxAge` : 쿠키 유효기간 설정
- `resave` : 세션에 변경사항이 없어도 요청마다 세션을 다시 저장, 기본 옵션인 true는 deprecated 상태로 false 권장
- `saveUninitialized` : 세션에 저장할 내용이 없더라도 uninitialized 상태의 세션을 저장, 기본 옵션인 true는 deprecated 상태로 false 권장

> `cookie`의 기본 값은  
> `{ path: '/', httpOnly: true, secure: false, maxAge: null }`

```javascript
// auth/session.js

import session from "express-session";
import dotenv from "dotenv";

dotenv.config();

export default (app) => {
  app.use(
    session({
      secret: process.env.SESSION_SECRET,
      cookie: { maxAge: 1000 * 60 * 60 }, // 1 hour
      resave: false,
      saveUninitialized: false,
    })
  );
};
```

### Passport 설정

코드 내의 주석으로 표기한 순서대로 이해하시면 좋을 것 같습니다.

1. `new LocalStrategy()` 를 통해 로컬 전략을 세웁니다.  
   소셜 로그인 인증을 위한 다른 전략을 세울 수도 있습니다.

   `usernameField`와 `passwordField`는 HTML form 내에 아이디, 비밀번호 관련 `<input name="" value="" />` 태그의 name 값을 넣어줍니다.

   이후 콜백 함수에서 해당 태그들의 value를 인자 값으로 전달받아  
   DB에 접근하여 요청한 정보가 일치하는 지 검증하는 로직을 수행합니다.  
   예제에서는 `sequelize`를 사용했습니다.

   만약, 일치한다면 `done()`을 통해 `passport.serializeUser()`로  
   `user` 를 전달합니다.

   DB를 설계한 방식에 따라 속성은 다를 수 있습니다.  
   ex) `user = { id: '', password: '', email: '' }`

2. `passport.serializeUser()` 함수를 통해 사용자 정보를 세션에 저장합니다.
   앞의 `LocalStrategy` 객체의 콜백 함수에서 `done()`으로 전달받은 `user`의 `id`를 세션에 저장하기 위해 `user.id`를 반환합니다.

3. `passport.deserializeUser()` 함수는 로그인 인증이 되어있는 경우, 요청할 때마다 호출하여 실행합니다.  
   이는 세션에 저장하려는 사용자 정보가 큰 경우, 메모리가 많이 소모되므로,  
   `passport.serializeUser()`에서 사용자의 키 값인 `id`만을 세션에 저장해 두고,  
   세션에 저장된 `id`를 이용하여 DB에 접근하여 사용자 정보를 추가로 `select` 한 후  
   `HTTP request` 객체에 붙여서 `req.user`를 반환합니다.

이를 정리하면 `serializeUser()`는 세션에 사용자 키 값 `id`만 저장하고,  
`deserializeUser()`는 세션에 저장된 `id`를 이용해서, 매번 DB에 추가로 필요한 사용자 정보를 `select`하여 `req.user`를 반환합니다.

```javascript
// auth/passport.js

import * as passport from "passport";
import { Strategy as LocalStrategy } from "passport-local";
// import { Strategy as NaverStrategy } from "passport-naver";
// import { Strategy as KakaoStrategy } from "passport-kakao";

import { User } from "../db/models";

export default (app) => {
  app.use(passport.initialize());
  app.use(passport.session());

  // (2) - 로그인 인증 성공 시 한 번만 실행
  passport.serializeUser((user, done) => {
    done(null, user.id); // 세션에 사용자 정보인 id를 저장
  });
  // (3) - 로그인 인증이 되어있는 경우, 요청할 때마다 실행
  passport.deserializeUser(async (id, done) => {
    // 세션에 저장되어 있는 id를 인자 값으로 전달받음
    try {
      const user = await User.findOne({
        where: {
          id,
        },
        raw: true,
      });
      done(null, user); // req.user 객체 생성
    } catch {
      done(null, false, {
        message: "서버에 문제가 발생했습니다. 잠시 후 다시 시도해 주세요.",
      });
    }
  });

  // (1)
  passport.use(
    new LocalStrategy(
      {
        session: true, // 세션 저장 여부
        usernameField: "id", // form > input name
        passwordField: "password",
      },
      async (id, password, done) => {
        try {
          // 회원정보 조회
          const user = await User.findOne({
            where: {
              email: id,
            },
            raw: true,
          });

          // 회원정보가 없거나 비밀번호가 일치하지 않는 경우
          if (!user || user.password !== password) {
            done(null, false, {
              message:
                "존재하지 않는 아이디이거나 비밀번호가 일치하지 않습니다.",
            });
          } else {
            done(null, user); // serializeUser로 user 전달
          }
        } catch {
          done(null, false, {
            message: "서버에 문제가 발생했습니다. 잠시 후 다시 시도해 주세요.",
          });
        }
      }
    )
  );
};
```

## 로그인 및 로그아웃 API

앞에서 진행했던 프로젝트 파일 구조에서 `src/` 내에 `controllers/`, `middlewares/`, `routes/`를 추가하여 각 디렉토리에 `auth.js`를 생성합니다.

> API 요청에 대한 라우팅 로직은 `routes/`에서만, 비즈니스 로직은 `controllers/`로 분리함으로써 가독성과 유지보수에 용이해집니다.

```
|-- src
     |-- auth
     |   `-- passport.js
     |   `-- session.js
     |-- controllers
     |   `-- auth.js
     |-- middlewares
     |   `-- auth.js
     |-- routes
     |   `-- auth.js
     |-- app.js
|-- .env
```

### 로그인 인증 여부 확인

로그인 인증 여부 확인을 위한 미들웨어 함수를 `middlewares/` 내에 `auth.js`에 추가합니다.  
`req.isAuthenticated()` 함수는 로그인이 되어있는 경우, `true`를 반환합니다.

따라서, 로그인이 필요한 요청과 필요하지 않은 요청에 대하여 각 미들웨어를 사용하면 됩니다.  
그리고 `next()`를 통해 다음 미들웨어 함수 로직을 수행하도록 합니다.

예를들어, 마이페이지 내 정보 조회와 같이 사용자 로그인이 필요한 경우 `isLoggedIn`를  
이미 로그인이 되어있는데 로그인 페이지에 접근한 경우 `isNotLoggedIn`를 사용합니다.

```javascript
// middlewares/auth.js

export const isLoggedIn = (req, res, next) => {
  if (req.isAuthenticated()) {
    next();
  } else {
    res.redirect("/auth/login");
  }
};

export const isNotLoggedIn = (req, res, next) => {
  if (!req.isAuthenticated()) {
    next();
  } else {
    res.redirect("/");
  }
};
```

### 로그인 및 로그아웃 구현

로그인 및 로그아웃 관련 라우팅 로직을 `routes/` 내에 `auth.js`에 추가합니다.  
앞에서 작성한 로그인 인증 여부 확인 미들웨어를 `import`하여  
`router.get("/", middleware)` 와 같이 미들웨어 함수를 사용할 수 있습니다.

그리고 로그인 인증 요청에 대한 라우팅은 `passport.authenticate()` 미들웨어 함수를 사용합니다. 이는 `'local'` 인자 값을 주어 앞에서 작성한 `new LocalStrategy()` 를 통해 세운 로컬 전략의 함수를 호출합니다.

> `'local'`이 아닌 소셜 로그인 관련 전략을 세웠다면,  
> `'kakao'`, `'naver'`를 인자 값으로 넘겨줍니다.

- `successRedirect` : 성공 시 redirect 경로
- `failureRedirect` : 실패 시 redirect 경로
- `successFlash`: 성공 시 출력할 flash 메세지를 직접 설정
- `failureFlash` : 실패 시 flash 메세지 여부, 로컬 전략의 콜백 함수에서 받은 메세지를 사용해 에러 메세지를 출력 또는 출력할 flash 메세지를 직접 설정

flash 메세지는 기본적으로 1회용 메세지이며, 세션에 저장해 두었다가 반환하게 됩니다.

> flash 메세지를 사용하려면 `req.flash()` 기능이 필요합니다. Express 2.x는 이 기능을 제공했지만 Express 3.x에서는 제거되어, [connect-flash](https://github.com/jaredhanson/connect-flash) 사용이 필요합니다.
>
> ```bash
> $ npm install connect-flash
>
> // app.js
>
> import flash from 'connect-flash'
>
> app.use(flash())
> ```

```javascript
// routes/auth.js

import express from "express";
import passport from "passport";

import authController from "../controllers/auth";
import { isLoggedIn, isNotLoggedIn } from "../middlewares/auth";

const router = express.Router();

/* /auth */
router.get("/login", isNotLoggedIn, authController.displayLogin);

router.post(
  "/login",
  isNotLoggedIn,
  passport.authenticate("local", {
    successRedirect: "/",
    failureRedirect: "/auth/login",
    failureFlash: true,
  })
);

router.get("/logout", isLoggedIn, authController.logout);

export default router;
```

다음으로 비즈니스 로직을 구현하기 위해 `controllers` 내에 `auth.js`를 작성합니다.  
이는 앞에서 `import`하여 사용할 `authController` 입니다.

```javascript
// controllers/auth.js

import express from "express";

const displayLogin = (req, res) => {
  res.render("login", { message: req.flash("error") });
};

const logout = (req, res) => {
  req.session.destroy();
  res.redirect("/");
};

export default {
  displayLogin,
  logout,
};
```

> 로그인하는 비즈니스 로직도 `controller`의 미들웨어로 분리할 수 있지 않나요?
>
> -> `passport.authenticate`는 이미 미들웨어이므로, 분리를 하게 되면 작동이 안되는 이슈가 발생했습니다. 이유는 미들웨어를 미들웨어로 한번 더 감싸서라고 추측하고 있습니다.

# References

- [dotenv](https://github.com/motdotla/dotenv)
- [express-session](https://github.com/expressjs/session)
- [Passport.js](http://www.passportjs.org/docs/)
