- Go to https://developer.spotify.com/dashboard/applications
- `Create an App`
- Edit Settings
- On Redirect URLs add `http://localhost:3000/users/auth/spotify/callback` and save
- Add your client ID and client Secret to your `.env`. Your .env should be like this

```
SPOTIFY_ID=db******c49
SPOTIFY_PWD=a2******58
```

[Check here](pictures/) the screenshots.
