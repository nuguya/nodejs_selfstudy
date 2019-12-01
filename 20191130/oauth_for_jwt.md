# Google Oauth for JWT

## using Oauth with passport
구글 클라우드 플랫폼에 들어가서 프로젝트 등록 후 OAuth key와 id를 발급받는다.
    - 이 때 개발용으로 사용했기에 url과 리다이렉션 url은 로컬호스트를 적용함.
    
등록받은 key와 id를 환경변수나 별도의 파일로 분리하여 저장.

passport-google-oauth20 모듈 설치 후 initialize를 위한 strategy 설정.

``` javascript
import passport from "passport";
import { Strategy } from "passport-google-oauth20";
import loadConfig from "../config/configLoader";
import { createHost, findHostById } from "../../DB/queries/host";

const GoogleStrategy = Strategy;

function extractProfile(profile) {
	let imageUrl = "";
	if (profile.photos && profile.photos.length) {
		imageUrl = profile.photos[0].value;
	}
	return {
		id: profile.id,
		displayName: profile.displayName,
		image: imageUrl,
		email: profile.emails[0].value,
	};
}

export default (function() {
	const { oAuthArgs } = loadConfig();
	passport.use(
		new GoogleStrategy(
			{ ...oAuthArgs },
			async (accessToken, refreshToken, profile, cb) => {
				try {
					const { id, displayName, image, email } = extractProfile(
						profile
					);
					let host = await findHostById(id);
					if (!host) host = await createHost(id, displayName, email);
					return cb(null, host);
				} catch (error) {
					console.error(error);
				}
			}
		)
	);
})();

```
* 따로 Token을 생성하기 때문에 accessToken과 refreshToken은 사용하지 않음, profile은 구글 인증 성공 후 가져올 수 있는 사용자 프로필에 해당.
* profile에서 반환하는 id 값으로 DB에 이미 등록되어있는지 체크 후 등록되어있지 않으면, 유저 생성.
* session을 사용하지 않기 때문에 serialize와 deserialize는 선언하지 않음.

구글 로그인에 접속 할 수 있는 endpoint를 정의하고 routing

``` javascript
import express from "express";
import passport from "passport";
import { generateAccessToken } from "../authentication/token";

const router = express.Router();

router.get(
	"/login",
	passport.authenticate("google", {
		session: false,
		scope: ["email", "profile"],
		prompt: "select_account",
	})
);

router.get("/logout", function(req, res, next) {
	req.logOut();
	res.redirect("/");
});

router.get(
	"/google/callback",
	passport.authenticate("google", {
		session: false,
	}),
	(req, res) => {
		const accessToken = generateAccessToken(req.user.oauthId);
		res.cookie("vaagle", accessToken);
		res.redirect("http://localhost:3002/");
	}
);
module.exports = router;

```
* req.user로 앞서 정의한 google strategy에서의 cb return 값을 읽어올 수 있음.
* passport.authenticate의 session 옵션을 명시적으로 false로 줌.
* promt option은 로그인 리다이렉션 시 자동로그인이 아닌 로그인 계정을 선택할 수 있는 창을 불러오도록 설정.


구글 로그인을 통해 사용자가 인증되면 JWT를 생성 후 클라이언트 쿠키에 등록시킨다.
이 후 토큰을 발급받은 클라이언트들을 passport jwt 전략을 통해 토큰 값을 복호시키고 유저가 있는지 확인하는 방식으로 인증을 수행한다.

``` javascript 
import passport from "passport";
import passportJwt from "passport-jwt";
import loadConfig from "../config/configLoader";
import { findHostById } from "../../DB/queries/host";

export default (function() {
	const { tokenArgs } = loadConfig();
	const jwtOptions = {
		jwtFromRequest: passportJwt.ExtractJwt.fromAuthHeaderAsBearerToken(),
		secretOrKey: tokenArgs.secret,
		issuer: tokenArgs.issuer,
		audience: tokenArgs.audience,
	};
	passport.use(
		new passportJwt.Strategy(jwtOptions, async (payload, cb) => {
			try {
				const host = findHostById(payload.sub);
				if (host) {
					return cb(null, host, payload);
				}
				return cb();
			} catch (error) {}
		})
	);
})();

```
* jwtFromRequest은 클라이언트 헤더에 담긴 jwt 토큰을 의미.
* payload는 토큰을 JWT payload 부분에 해당.

### RefreshToken ??
만약 RefreshToken을 사용한다면 accessToken은 클라이언트 inMemory에 저장하고 refreshToken은 cookie에 저장하여 사용하도록 할 예정.

## 프로젝트 적용
![diagram](https://github.com/nuguya/nodejs_selfstudy/blob/master/20191130/%EB%A1%9C%EA%B7%B8%EC%9D%B8%20diagram.png)

- [spa-social-login-google-facebook](https://www.sitepoint.com/spa-social-login-google-facebook/)