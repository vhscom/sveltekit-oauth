[![Latest NPM version](https://flat.badgen.net/npm/v/sveltekit-oauth)](https://npmjs.com/sveltekit-oauth)
[![Total Downloads](https://flat.badgen.net/npm/dt/sveltekit-oauth)](https://npmjs.com/sveltekit-oauth)
[![Minimum SvelteKit version](https://flat.badgen.net/badge/SvelteKit/>=1.0.0-next.286/ff3e00)](https://github.com/sveltejs/kit/blob/master/packages/kit/CHANGELOG.md#100-next286)
![Type Definitions](https://flat.badgen.net/npm/types/sveltekit-oauth)
[![MIT licensed](https://flat.badgen.net/npm/license/sveltekit-oauth)](https://codeberg.org/vhs/sveltekit-oauth/src/branch/trunk/LICENSE)

# SvelteKit OAuth

> Based on the original SvelteKitAuth https://github.com/Dan6erbond/sk-auth

SvelteKit authentication library with built-in OAuth providers and unrestricted customization.

## Installation

Install using your preferred package manager:

```bash
pnpm add -D sveltekit-oauth # or yarn add, npm install, etc.
```

### Usage with TypeScript

SvelteKit OAuth also comes with first-class support for TypeScript out of the box, so no need to add an additional `@types/` dev dependency! 🎉

## Getting Started

SvelteKit OAuth is very easy to setup! All you need to do is instantiate the `SvelteKitAuth` class, and configure it with some default providers, as well as a JWT secret key used to verify the cookies:

***Warning**: env variables prefixed with `VITE_` can be exposed and leaked into client-side bundles if they are referenced in any client-side code. Make sure this is not the case, or consider using an alternative method such as loading them via dotenv directly instead.*

```ts
export const appAuth = new SvelteKitAuth({
  providers: [
    new GoogleOAuthProvider({
      clientId: import.meta.env.VITE_GOOGLE_OAUTH_CLIENT_ID,
      clientSecret: import.meta.env.VITE_GOOGLE_OAUTH_CLIENT_SECRET,
      profile(profile) {
        return { ...profile, provider: "google" };
      },
    }),
  ],
  jwtSecret: import.meta.env.JWT_SECRET_KEY,
});
```

If you want to override or augment the default SvelteKit session to get access to the user in the `session` store, you can use the `getSession` hook:

```ts
// overriding the default session
export const { getSession } = appAuth;

// augmenting it
export const getSession: GetSession = async (request) => {
  const { user } = await appAuth.getSession(request);

  return { user };
};
```

## Callbacks

SvelteKit OAuth provides some callbacks, similar to NextAuth.js. Their call signatures are:

```ts
interface AuthCallbacks {
  signIn?: () => boolean | Promise<boolean>;
  jwt?: (token: JWT, profile?: any) => JWT | Promise<JWT>;
  session?: (token: JWT, session: Session) => Session | Promise<Session>;
  redirect?: (url: string) => string | Promise<string>;
}
```

## Adding more Providers

SvelteKit OAuth uses a object-oriented approach towards creating providers. It is unopinionated and allows you to implement any three-legged authentication flow such as OAuth, SAML SSO, and even regular credential logins by omitting the `signin()` route.

You can implement your own using the `Provider` base provider class, and by implementing the `signin()` and `callback()` methods:

```ts
export abstract class Provider<T extends ProviderConfig = ProviderConfig> {
  abstract signin<Locals extends Record<string, any> = Record<string, any>, Body = unknown>(
    request: ServerRequest<Locals, Body>,
  ): RequestHandlerOutput | Promise<RequestHandlerOutput>;

  abstract callback<Locals extends Record<string, any> = Record<string, any>, Body = unknown>(
    request: ServerRequest<Locals, Body>,
  ): CallbackResult | Promise<CallbackResult>;
}
```

`signin()` must return a generic endpoint output, this can be a redirect, or the path to the provider's sign-in page. When implementing a `HTTP POST` route, `signin()` can simply return an empty body and `callback()` should handle the user login flow.

`callback()` takes a `ServerRequest` and must return a `CallbackResult` which is a custom type exported by `sveltekit-oauth`:

```ts
export type Profile = any;
export type CallbackResult = [Profile, string | null];
```

The first item in the tuple is the user profile, which gets stored in the token, and is provided to the `jwt()` callback as the second argument. The second item is a redirect route, which may be tracked using the `state` query parameter for OAuth providers, or other implementations depending on the sign-in method.

### OAuth2

SvelteKitAuth comes with a built-in OAuth2 provider that takes extensive configuration parameters to support almost any common OAuth2 provider which follows the OAuth2 spec. It can be imported from `sveltekit-oauth/providers` and configured with the following configuration object:

```ts
export interface OAuth2ProviderConfig<ProfileType = any, TokensType extends OAuth2Tokens = any>
  extends OAuth2BaseProviderConfig<ProfileType, TokensType> {
  accessTokenUrl?: string;
  authorizationUrl?: string;
  profileUrl?: string;
  clientId?: string;
  clientSecret?: string;
  scope: string | string[];
  headers?: any;
  authorizationParams?: any;
  params: any;
  grantType?: string;
  responseType?: string;
  contentType?: "application/json" | "application/x-www-form-urlencoded";
}
```

Some values have defaults which can be seen below:

```ts
const defaultConfig: Partial<OAuth2ProviderConfig> = {
  responseType: "code",
  grantType: "authorization_code",
  contentType: "application/json",
};
```

The `OAuth2Provider` class can then be instantiated with the configuration to support the OAuth2 flow, including authorization redirect, token retrieval and profile fetching. It will also automatically handle the `state` and `nonce` params for you.

## Motivation

SvelteKit OAuth is inspired by the [NextAuth.js](https://next-auth.js.org/) package built for the Next.js SSR framework for React. Unlike NextAuth.js it is completely unopinionated and only provides implementations for default flows, while still empowering users to add their own providers.

As it leverages classes and TypeScript, the implementation of such providers is very straightforward, and in the future it will even be possible to register multiple SvelteKit OAuth handlers in the same project, should the need arise, by leveraging a class-based client and server setup.

## Examples

See the [example app](./app/) in the repository source.

## Rights

This project is licensed under the terms of the MIT license.
