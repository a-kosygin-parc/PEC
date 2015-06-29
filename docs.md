#Документация

## Запуск сервера 

Требуется Node.js > 0.10.5

```shell
npm install #устанавливает пакеты

#запусткает локальный сервер на указанном порте (3002) по-умолчанию
node node/app.js -p=[port_number] 
```
Чтобы выводил лог надо переправить [здесь все поля на true](https://github.com/d-mkrtchyan-parc/PEC/blob/master/node/config/config.js#L44)

### Разработка

Стили и JS собирается с помощью gulp, поэтому, чтобы изменения вошли в силу, нужно либо запустить gulp.watch
```
$ gulp
```
Который будет следить за изменениями, либо запустить задачи-сборщики вручную:
```
$ gulp styles
$ gulp scripts
```

### Блоки и данные

Изначальный список полей подгружается отсюда: [public/controllers/api/fields.json](https://github.com/d-mkrtchyan-parc/PEC/blob/master/public/controllers/api/fields.js)

В нем перечисленны блоки, их типы и модель данных. Модель данных связывается следующим образом. Например, объект блока:
Например:
```js
{
  "index":4, // порядок в списке
  "id":"417-756-345", // уникальный идентификатор (неважен)
  "type":"r", // тип (r - это ribbon)
  "template":"jade/template/ribbon.tpl", // URL шаблона
  "form":false, // можно ли конфигурировать данные
  "repeat":false, // можно ли повторять
  "quantitable":true, // можно ли изменять число (единственное/множественное)
  "quantity":0, // число (1 - единственное)
  "disabled":false, // выключенно
  "name":"Лента: акция", // имя блока
  "data":{ // данные, которые будут интерполированны в шаблон
    "singular":{
      "number":5,
      "name":"Лента: акция",
      "width":192,
      "height":36 },
    "plural":{
      "width":192,
      "height":35,
      "number":7,
      "name":"Лента: акции"}},
  "hidden":true // скрыт
  }
  ```
  
И есть шаблон, соответствующий этому блоку в `jade/template/ribbon.tpl`, в котором ссылка на field - это ссылка на объект блока.
```jade
// Ленточка: {{field.data[field.quantity ? 'plural' : 'singular' ].name}} 
div(style="...")
	img(ng-src="{{field.data[field.quantity ? 'plural' : 'singular'].number}}.png" 
	    width="{{field.data[field.quantity ? 'plural' : 'singular'].width}}" 
	    height="{{field.data[field.quantity ? 'plural' : 'singular'].height}}" 
	    alt="{{field.data[field.quantity ? 'plural' : 'singular'].name}}")
//-...
```
Внутри шаблона, можно интерполировать данные из объекта. Этим занимается Angular.js (см. [правила выражений](https://docs.angularjs.org/guide/expression))

Если данные динамические, и конфигурируются через конструктор, то св-во `form` должно быть равно true, а форма для связывания данных с моделью будет лежать в папке jade/forms/{{type}}.


## Компиляция

Компиляцией занята директива [letter](https://github.com/d-mkrtchyan-parc/PEC/blob/master/public/static/js/modules/ng/directives.js#L50).
Директива вычищает (или должна вычищать) все следы angular (ng-атрибуты и комментарии).

## Аутентификация

Принцип обычной HTTP-AUTH через пост-запрос, хэширования и шифрования нет.
CVS-файл c парами логин/пароль лежит в [node/users.htpasswd](https://github.com/d-mkrtchyan-parc/PEC/blob/master/node/users.htpasswd)

## Стэк
