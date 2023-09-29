# trueconf

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run start
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).


<!--  -->

Кнопка на этаже: 
    красная - лифт едет на этаж
    желтая - этаж в очереди или добавлен в нее

Вызов пропускается в случаях, если:
- лифт уже находится на выбранном этаже в состоянии покоя

- лифт уже находится в процессе обработки такого вызова (находится в
движении к выбранному этажу)

- в очереди вызовов уже есть выбранный этаж

Если на этаже есть лифт который на паузе то вызывается другой свободный лифт

Демонстрация работы - http://lifts.z-email.ru/