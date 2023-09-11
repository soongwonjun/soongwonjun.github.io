---
layout: post
title:  스키마를 검증하는 방법. Joy & Ajv in NestJs
date:  2023-06-01 22:00:00 +0900
categories: nodejs
tags: nodejs cheatsheet
---

## Joy

```typescript
@Injectable()
export class JoiValidationPipe implements PipeTransform {
  constructor(private schema: ObjectSchema) {}

  transform(value: any, metadata: ArgumentMetadata) {
    const result = this.schema.validate(value);
    if (result.error) {
      throw new HttpException(
        {
          message: 'Validation failed',
          detail: result.error.message.replace(/"/g, `'`),
          statusCode: HttpStatus.BAD_REQUEST,
        },
        HttpStatus.BAD_REQUEST,
      );
    }
    return value;
  }
}

@UsePipes(
  new JoiValidationPipe(
    Joi.object({
      accountId: Joi.string().required(),
      token: Joi.string().required(),
      type: Joi.string().required().valid(AccountTypes),
    }),
  ),
)
```

## Ajv

```typescript
export class AjvValidationPipe implements PipeTransform {
  private ajv;
  constructor(private schema: AnySchema) {
    this.ajv = new Ajv();
  }

  transform(value: any, metadata: ArgumentMetadata) {
    const validate = this.ajv.compile(this.schema);
    if (!validate(value)) {
      throw new HttpValidationException(validate.errors);
    }
  }
}

@UsePipes(
  new AjvValidationPipe({
    type: 'object',
    properties: {
      accountId: { type: 'string' },
      token: { type: 'string' },
      type: { type: 'string' },
    },
    required: ['acocunt', 'token', 'type'],
  }),
)
```

## Reference

- [Joi](https://github.com/hapijs/joi#readme)
- [Ajv](https://ajv.js.org/)
- [Ajv - Git](https://github.com/ajv-validator/ajv)
