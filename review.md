### Исходный код № 1 (файл Header.tsx)
``` typescript
import Avatar from "@components/Avatar";
import { useEffect, useState } from "react";
import UserAPI from "@api/UserAPI";
  
export default function Header() {
  const [user, setUser] = useState(null);

  useEffect(() => {
    UserAPI.fetchUser().then((userData) => {
      setUser(userData);
    });
  });
  
  return (
    <div className="Header">
      <Avatar user={user} />
    </div>
  );
}
```

#### Ревью (Header.tsx)

1. Рекомендуется отказаться от использование экспортов по умолчанию в пользу именованных экспортов, так как при увеличении проекта они будут вносить больше запутанности, поскольку они не предоставляют явных указателей на то, что и откуда экспортируется,
2. `useEffect` без массива зависимостей. Здесь использован бесконечный `useEffect`, так как вторым параметром не передан массив зависимостей. Это означает, что `useEffect` будет выполняться при каждом рендеринге компонента. Вместо этого необходимо передать пустой массив зависимостей, чтобы `useEffect` выполнялся только при монтировании компонента.
  ``` typescript
  useEffect(() => {
    UserAPI.fetchUser().then((userData) => {
      setUser(userData);
    });
  }, []); // Пустой массив зависимостей
```
3.  Нет обработки ошибок при запросе данных с сервера - Код не обрабатывает возможные ошибки при выполнении запроса к `UserAPI.fetchUser()`. Необходимо обрабатывать ошибки для try/catch, чтобы мы могли контролировать это поведение.
   
```typescript
useEffect(() => {
    const fetchUserData = async () => {
      try {
        const userData = await UserAPI.fetchUser();
        setUser(userData);
      } catch (error) {
        console.error('Error fetching user data:', error);
        // Обработка ошибки
      }
    };
    fetchUserData();
  }, []);
```
4. Нет фоллбека на то, что данные в user могут не прийти и наши данные остануться `null`,
 Здесь лучше воспользоваться опциональным рендерингом и отдавать пропсами внутрь `Avatar` только данные, которые имеют значения, иначе совсем не рендерить его

```typescript
{user && <Avatar user={user} />}
```
5. Импорты `Avatar.tsx` на верхнем файле лежат импорты из react на самом верху, в текущем - нет, необходимо привести в соответствие и располагать импорты из react на самом верхнем уровне.
### Исходный код №2 (файл Avatar.tsx)
```typescript

import React, { FC, useMemo, ImgHTMLAttributes } from 'react';
import "./avatar.scss";
const avatarTmpl = require('@root/images/no-avatar.png').default;

type Props = {

  onInputChange: (e: React.ChangeEvent<HTMLInputElement>) => void

} & ImgHTMLAttributes<HTMLImageElement>;

const Avatar: FC<Props> = ({
  src = avatarTmpl,
  onInputChange = undefined,
  user,
  ...rest
}) => {
  const { firstName, lastName } = user;
  const fullName = useMemo(() => `${firstName} ${lastName}`);
  
  return (

    <div className="profile-avatar">
      <input
        type="file"
        className="profile-avatar__input"
        onChange={(event) => {
          onInputChange(event);
        }}
      />
      <span>{fullName}</span>
      <img
        className="profile-avatar__image"
        src={src}
        alt={rest.alt}
      />
    </div>
  )
};

export default Avatar;

```

#### Ревью (Avatar.tsx)

1. Нет необходимости импортировать тип `ChangeEvent` из `React.ChangeEvent`, можно просто сделать `import {ChangeEvent} from 'react';`
2. Отступы между импортами убрать и привести в соответствие с файлом `Header.tsx`, там импорты без отступов между ними;
3. Не определен тип пропса `user`, поэтому его необходимо добавить в тип  `Props` и выше объявить `type User = {firstName: string, lastName: string}`;
4. Нет необходимости расширять наш тип для `Avatar` через `ImgHTMLAttributes<HTMLImageElement>;`, так как мы используем из него только `alt` и `src`, следовательно такое расширение для нас избыточно, также и использование `rest` также будет уже не акутальным; 
5.  `onInputChange` - не определен в типах как опциональный, а сразу сетится `undefined`, лучше в типах указать его опциональным;
   
   
   в конечном итоге тип `Props` у нас должен будет выглядеть таким образом
```typescript
type Props = {
  onInputChange?: (e: React.ChangeEvent<HTMLInputElement>) => void;
  alt?: string;
  src?: string;
}
```

6.  Необходимо привести в соответствие импорты для `avatarTmpl` и отказаться от использования `requaire` с дефолтным экспортов
   В современных приложениях, особенно в проектах, использующих синтаксис ECMAScript Modules (ESM) (например, с использованием `import` и `export`), предпочтительнее использовать синтаксис `import` вместо `require`. 
```typescript
import avatarTmpl from '@root/images/no-avatar.png';
```
Это синтаксически более современный и читаемый подход. 

7. Лишнее использование `useMemo`:
   В данном случае использование `useMemo` избыточно, так как `fullName` не используется внутри рендера какого-либо условия. Просто необходимо установить `fullName` без `useMemo`;
```typescript
const fullName = `${firstName} ${lastName}`;
```
Также хорошей практикой является выносить подобные чистые функции в utils, если вдруг это где-то еще нам пригодится
```typescript
const getFullName = (firstName: string, lastName: string): string => `${firstName} ${lastName}`
```
8.  Здесь также рекомендуется отказаться от использование экспортов по умолчанию в пользу именованных экспортов, так как при увеличении проекта они будут вносить больше запутанности, поскольку они не предоставляют явных указателей на то, что и откуда экспортируется.
9. Нет необходимости создавать анонимную функцию внутри `onChange` можем сделать просто `onChange={onInputChange}`
10. Отсутствие поддержки разных типов файлов - если `input` необходим для загрузки изображений, нужно указать ему атрибут `accept` с соответствующим значением, чтобы была возможность ограничить типы загружаемых файлов