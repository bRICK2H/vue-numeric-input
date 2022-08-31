# input-numeric

## Config:
| Name 				  	| Type 					      | Default 			  	| Description
| ---- 				  	| :--: 					      | :-----: 			  	| ---------------
| value 		  			| `[String, Number]`			| **0** 		  			| Реактивное значение
| separator 		  	| `Boolean`						| **true** 		  		| Разделить числа по их значению вида 1 000, если false то 1000
| decimal 		  		| `[String, Number]`			| **0** 		  			| Колличество знаков после запятой
| min 		  			| `[String, Number]`			| **0** 		  			| Минимальное допустимое значение
| max 		  			| `[String, Number]`			| **100000** 		  	| Максимальное допустимое значение
| isDefaultReset 		| `Boolean`	          		| **true** 				| Сбрость значение по Escape на значение по умолчанию (инициализированное)
| disabled 				| `Boolean`	          		| **false** 			| Заблокировать поле ввода
| focusSelected 		| `Boolean`	          		| **true** 				| Включить фокисировку
| focusOnOpen 			| `Boolean`	          		| **false** 			| Включить фокисировку при инициализации компонента
| toggleFocus 			| `Boolean`	          		| **false** 			| Манипулирование фокусом за счет параметра
| width 					| `[String, Number]`			| **150** 			   | Ширина инпута
| height 			  	| `[String, Number]`			| **37** 			   | Высота инпута
| discount 			  	| `Boolean`						| **false** 			| Включить шаблон выбора скидки
| updateButton 		| `Boolean`						| **false** 			| Включить кнопку "обновить"
| currency 			  	| `String`						| **df** 			   | Подключение валюты, соществующие { ru: рубль, lb: фунт, en: доллар, df: '' }
| target 			  	| `String`						| **''** 			   | Название класса, присваивается для элемента обертки input

## Events:

```javascript
// Возвращает введенное значение
$emit('input', value)

// Возвращает event перед уничтожением
$emit('beforeClose')

// Возвращает событие потери фокуса
$emit('blur')

/**
 * if discount: true
 * Возвращает объект - значение и тип скидки
 */
$emit('input', { value, type })

/**
 * if updateButton: true
 * Возвращает событие update:value
 */
$emit('update:value')
```