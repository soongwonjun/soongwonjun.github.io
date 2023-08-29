---
layout: post
title:  SNS 인증을 위한 코드 모음
date:  2023-06-01 22:00:00 +0900
categories: nodejs
tags: nodejs cheatsheet
description: nodejs에서 apple/google/kakao 인증 연동을 만들어봅시다.
---

## Abstract Class

```ts
export abstract class SocialAuthService {
  constructor() {}

  async verifyUser(token: string): Promise<void> {
    try {
      await this.verifyToken(token);
      return;
    } catch (e) {
      throw new Error('unauthorized')
    }
  }

  protected abstract verifyToken(token: string): Promise<any>;
}
```

## Apple

```ts
export class SocialAuthAppleService extends SocialAuthService {
  protected async verifyToken(token: string): Promise<any> {
    const { data } = await axios.post(
      'https://appleid.apple.com/auth/token',
      {
        client_id: 'CLIENT_ID',
        client_secret: 'CLIENT_SECRET',
        grant_type: 'authorization_code',
        code: token,
      },
      {
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded',
          Authorization: `Bearer ${token}`,
        },
      },
    );
    return {
      socialToken: data['id_token'],
      expire: data['expires_in'],
    };
  }
}
```

## Google

- Google은 NodeJS를 위한 라이브러리 `google-auth-library`를 제공한다.

```ts
export class SocialAuthGoogleService extends SocialAuthService {
  private client: OAuth2Client;

  constructor() {
    super();
    const clientId = ''; // Google의 ClientId
    this.client = new OAuth2Client(clientId);
  }

  protected async verifyToken(token: string): Promise<any> {
    const ticket = await this.client.verifyIdToken({
      idToken: token,
    });
    const payload = ticket.getPayload();
    return {
      socialToken: payload['sub'],
      email: payload['email'],
      expire: payload['exp'],
    };
  }
}
```

## Kakao

```ts
export class SocialAuthKakaoService extends SocialAuthService {
  protected async verifyToken(token: string): Promise<any> {
    const { data } = await axios.get('https://kapi.kakao.com/v1/user/access_token_info', {
      headers: {
        Authorization: `Bearer ${token}`,
      },
    });
    return {
      socialToken: data['id'],
      expire: data['expires_in'],
    };
  }
}
```

## Appendix

- [Apple API](https://developer.apple.com/documentation/sign_in_with_apple/generate_and_validate_tokens)
- [Google API](https://developers.google.com/identity/sign-in/web/backend-auth?hl=ko)
- [Kakao API](https://developers.kakao.com/docs/latest/en/kakaologin/rest-api#get-token-info)
