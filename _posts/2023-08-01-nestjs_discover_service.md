---
layout: post
title:  NestJS Discovery Service에 대해 알아봅시다
date:  2023-08-01 22:00:00 +0900
categories: nodejs
tags: nodejs nestjs cheatsheet
---

## 계기

프로젝트를 진행하면서 공통 작업들의 요구사항이 생기게 되었다. 캐싱, 로깅, 모니터링, 검증 및 데이터 가공과 같은 작업들이 추가되게 되었고, 각 작업들에 대해 공통 코드들이 만들어지게 되었다. 하지만 매번 이러한 서비스들에 대해 공통 함수를 호출하게 하는건 많은 보일러 플레이트를 만들게 되었다. 때문에 NestJs의 데코레이터 관리 기능을 사용하여 이 부분을 구현하고자 하였다.  
하지만 단순히 검증 및 데이터 조작 혹은 로깅까지는 Decorator 함수를 만들고 필요한 부분에 선언만 해주면 되었다. 하지만 캐싱, 이벤트 발송, 혹은 메트릭과 같은 작업들에 대해서는 특별한 NestJs 모듈이 필요하며, 스토리지 서버로의 연결과 같은 초기화 작업까지 필요하게 된다. 이 경우에는 단순히 Decorator 함수를 만드는 정도가 아닌 조금 더 손이 가는 작업을 해야 한다.  
이 페이지에서는 이 방법에 대해 간단하게 요약 및 정리를 해본다.

## Typescript Decorator에 대한 요약

매우 좋은 방법, 매우 강력한 방법이라고 하는데 이게 뭐길래 그렇게 대단하다는 말을 많이 쓸까?  
이유는 데코레이터를 사용하면 함수나 클래스에 영향을 주지 않으면서 데코레이터의 대상에게 함수 체인을 걸 수 있다. 즉 Pure function 혹은 First-class function을 조합하여 다양한 기능을 구현할 수 있으면서 대상이 되는 함수에 영향을 주지 않는다는 의미이다.
Typescript에서는 이 기능을 사용하여 shim을 구현할 수 있도록 해주며, 여기서는 class/method 데코레이터를 사용하였다. 단 아직 실험적인 기능이므로, 참고문서의 링크 페이지를 참고하여 `experimentDecorators`를 활성화 한 후 사용하자.

## 구현해 봅시다

- Decorator 함수에서는 NestJs의 SetMetadata를 사용해서 필요한 정보를 세팅한다.
- NestJs 서비스가 시작될 때, DiscoveryService를 사용하여 메타데이터를 추출하고, 필요한 Decorator를 붙인다.

### 먼저 Decorator 함수를 작성해봅시다

```ts
import { SetMetadata } from '@nestjs/common';
export const MY_CACHE_SYMBOL = 'custom/myCache';
export function MyCache = (options: CacheOptions = {}): ParamDecorator => 
  SetMetadata(MY_CACHE_SYMBOL, options);
```

### 모듈이 초기화될 때 필요한 서비스를 inject받아서 로직과 연결시켜봅시다

```ts
// my-cache.ts
// MyCache Module을 만들고 MyCacheService에서 discoverService를 사용해 모든 서비스들을 찾아내어 metadata를 확인하고 proxy를 만들어봅니다.

@Injectable()
export class MyCacheService implements OnModuleInit {
  constructor(
    private readonly storage: Redis,
    private readonly discoveryService: DiscoveryService,
    private readonly scanner: MetadataScanner,
    private readonly reflector: Reflector) {}

  onModuleInit() {
    this.discoveryService
      .getProviders()
      .filter((wrapper) => {
        if (!wrapper.isDependencyTreeStatic()) return false;
        if (!wrapper.metatype || !wrapper.instance) return false;
        return true;
      })
      .forEach(({ instance, name }) => {
        this.scanner.getAllMethodNames(instance).forEach((method) => {
          const metadata: CacheOptions = this.reflector.get(MY_CACHE_SYMBOL, instance[method]);
          if (!metadata) {
            return;
          }

          const originalMethod = instance[method];
          instance[method] = new Proxy(originalMethod, {
            apply: async (t, thisArg, fnArgs) => {
              const cache = await this.getValue(metadata);
              if (cache) return cache;

              const result = await originalMethod.apply(thisArg, fnArgs);
              this.setValue(key, result, ResultCacheTtl[metadata.unit]);
              return result;
            },
          });
        });
      });
  }
}

// app.module.ts
// 만들어진 MyCacheModule은 app에 추가해두도록 합시다.

@Module({
  imports: [
    ...
    MyCacheModule,
  ],
})
export class AppModule {}
```

### 이제 만들어진 Decorator를 붙여서 사용해봅시다

```ts
@Injectable()
export class FooService {
  constructor(private readonly fooRepository: FooRepository) {}

  @MyCache({ useKey: 'foo.daily', ttlUnit: CacheTtlUnit.Daily })
  findDailyCap() {
    return await this.fooRepository.findOne({
      where: {
        expireAt: LessThen(+new Date())
      }
    })
  }

}
```

## Appendix

- [Typescript Decorator](https://www.typescriptlang.org/docs/handbook/decorators.html)
- [NestJs Decorator](https://docs.nestjs.com/custom-decorators)
- [NestJs Discovery](https://github.com/nestjs/nest/tree/master/packages/core/discovery)
- [Javascript Reflect](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect)
