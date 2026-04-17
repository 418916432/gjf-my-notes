## Doc
### Google login
```
谷歌登录成功会在url上带上code=xxxx，请求端需要用这个code换取google的访问令牌
```
### exchange google token
```
async with httpx.AsyncClient() as client:
        token_resp = await client.post(
            GOOGLE_TOKEN_URL,
            data={
                "code": code,
                "client_id": settings.GOOGLE_CLIENT_ID,
                "client_secret": settings.GOOGLE_CLIENT_SECRET,
                "redirect_uri": settings.GOOGLE_REDIRECT_URI,
                "grant_type": "authorization_code",
            },
        )


```
### get google user info
```
async with httpx.AsyncClient() as client:
        response = await client.get(
            GOOGLE_USERINFO_URL,
            headers={"Authorization": f"Bearer {access_token}"},
        )
        
```
