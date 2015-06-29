#Документация

## Запуск сервера 

Требуется Node.js > 0.10.5. Сервер написан с помощью Express 4.

```shell
npm install #устанавливает пакеты

#запусткает локальный сервер на указанном порте (3002) по-умолчанию
node node/app.js -p=[port_number] 
```
Чтобы выводил лог надо переправить [здесь все поля на true](https://github.com/d-mkrtchyan-parc/PEC/blob/master/node/config/config.js#L44)

----- 

## Разработка

Стили и JS собирается с помощью gulp, поэтому, чтобы изменения вошли в силу, нужно либо запустить gulp.watch через
```
$ gulp
```
Который будет следить за изменениями, либо запустить задачи-сборщики вручную:
```
$ gulp styles
$ gulp scripts
```
### Jade

Вместо HTML используется [Jade](http://jade-lang.com/), т.к. он намного лаконичнее. 

Настраивается это в congih.json и [app.js](https://github.com/d-mkrtchyan-parc/PEC/blob/master/node/app.js#L44). Можно убрать, но тогда надо будет не рендерить Jade, а читать HTML файлы и отправлять их клиенту. Модифицировать нужно будет [все запросы типа `*.tpl`](https://github.com/d-mkrtchyan-parc/PEC/blob/master/node/app.js#L134)

Например:
```js
app.get('/*.tpl', function (req, res, next) {
  	var	originalUrl = req.params[0],
  		url = originalUrl.split('/'),
		name = url.pop(),
		resolveName, model, data, file;
	
	name = utils.retrive(url, name, "html");
	file = config.root + config.views + name + '.html';
	
	fs.readFile(file, {encoding: 'utf-8'}, function(err, data){
		if(err){
			res.send('Sorry, not fount.');
		}else{
			res.send(data);
		}
		next();
	});
});
```

**Вниманиe!!!** : Все ссылки на шаблоны внутри кода обозначены как `.tpl`, чтобы потом можно было выбирать движок шаблонов. Нельзя с текущим кодом сочетать **и HTML и Jade**. Для этого нужно написать отдельный express-handler, который будет анализировать запрос на предмет расширения файла. 

*Я рекомендую оставить Jade т.к. он сильно упрощает синтаксис верстки.* 

-----

## Управление констурктором

### Переменные шаблона

В самом [шаблоне письма](https://github.com/d-mkrtchyan-parc/PEC/blob/master/public/jade/table.jade) есть переменные, которые интерполируются прямо из контроллера `mainCtrl`, например, номер телефона. Или инициализируются прямо в шаблоне, например, [фон письма](https://github.com/d-mkrtchyan-parc/PEC/blob/master/public/jade/index.jade#L87). Добавить дополнительную переменную можно в контроллер `mainCtrl` и связать ее с шаблоном письма напрямую с помощью Mustache-синтаксиса.

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

### Компиляция

Компиляцией занята директива [letter](https://github.com/d-mkrtchyan-parc/PEC/blob/master/public/static/js/modules/ng/directives.js#L50).
Директива вычищает (или должна вычищать) все следы angular (ng-атрибуты и комментарии).

### Как добавить новый блок?

#### Статичный блок
Допустим нужно добавить статичный блок рекламы. Нужно завести файл в котором будет верстка этого блока, например:

Файл: jade/template/ad.jade
```jade
div.static-template
 p {{field.data.title ? field.data.title : "Заголовок-по-умолчанию"}}
 p {{field.data.message}}
```
Теперь чтобы блок подгружался нам нужно завести для него конфигурационный объект в fields.json:
```js
{
  "id" : <<ID>>,
  "name" : "Advertismen",
  "type":"t", // t-текст (влияет на цвет блока в констр-ре)
  "template":"jade/template/ad.tpl",
  "form": false,
  "repeat":true,
  "quantitable":false,
  "disabled":false,
  "data" : {
    "title" : "Заголовок блока",
    "message": "Реклама!"
   }
}
```

Блок готов. Для того, чтобы заводить похожие блоки нужно будет копировать их в fields.json и руками изменять данные в св-ве `data`. Очевидно это не очень удобно и приемлемо только для тех типов блоков, которые не имеют точной привязки для конфигурации и их значения проще захардкодить (например. ribbon).

####Динамический блок
Теперь сделаем блок динамически конфигурируемым. Для этого нужно сделать св-во `form` ранвым `true` и сверстать шаблон формы, в которой мы будет связывать представление с моделью. Создадим файл:

Файл: jade/forms/ad.tpl
```jade
div
	form
		input(	type="text", 
			ng-model="field.data.title",
			value="{{field.data.title}}",
			placeholder="Заголовок рекламы")
		input(	type="text", 
			ng-model="field.data.message",
			value="{{field.data.message}}",
			placeholder="Сообщение")
```

С помощью этой формы Angular.js будет связывать данные с этим представлением и теперь в конструкторе можно будет открыть эту форму кликом на название блока в списке блоков и задать для каждого блока свои значения данных прямо в редакторе.

### Аутентификация

Принцип обычной HTTP-AUTH через пост-запрос, хэширования и шифрования нет.
CVS-файл c парами логин/пароль лежит в [node/users.htpasswd](https://github.com/d-mkrtchyan-parc/PEC/blob/master/node/users.htpasswd)

-----

### Клиентский стэк и REST

Приложение на клиенте запускается в файле [launcher.js](https://github.com/d-mkrtchyan-parc/PEC/blob/master/public/static/js/app/launcher.js)

REST API реализован с помощью [Warden.js](https://github.com/zefirka/Warden.js) в файле [functional.js](https://github.com/d-mkrtchyan-parc/PEC/blob/master/public/static/js/modules/ng/functional.js#L58). Результаты записываются и читаются из файла `public/controllers/files/fields.json`.
