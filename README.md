# codecs
Implicits. JSON Codecs

Огляд
Метою цього завдання є реалізація бібліотеки серіалізації.
Формат серіалізації - JSON. Обов’язково ознайомтесь із цим форматом.

Щоб спростити задачу, кодери та декодери, реалізовані в цьому завданні, безпосередньо не працюють з BLOB-масивами Json, вони працюють з проміжним типом даних Json представленим наступним чином:

sealed trait Json
object Json {
 /** The JSON `null` value */
 case object Null extends Json
 /** JSON boolean values */
 case class Bool(value: Boolean) extends Json
 /** JSON numeric values */
 case class Num(value: BigDecimal) extends Json
 /** JSON string values */
 case class Str(value: String) extends Json
 /** JSON objects */
 case class Obj(fields: Map[String, Json]) extends Json
 /** JSON arrays */
 case class Arr(items: List[Json]) extends Json
}

Ось приклад значення JSON:
{
  "foo": 0,
  "bar": [true, false]
}

Це можна представити наступним чином із типом даних Json:


Json.Obj(Map(
  "foo" -> Json.Num(0),
  "bar" -> Json.Arr(Json.Bool(true), Json.Bool(false))
))
 
При визначенні цього типу Json кодування значення типу A полягає у перетворенні його у значення типу Json:

trait Encoder[-A] {

 def encode(value: A): Json
}

На відміну від довільних значень JSON, об’єкти JSON мають особливу властивість, що два об’єкти можна об’єднати для створення більшого об’єкта JSON, що містить поля обох об’єктів. Ми використовуємо цю властивість об'єднання об'єктів JSON для визначення методу zip на Encoder, що повертає об'єкти JSON. Ми робимо це в підкласі Encoder, який називається ObjectEncoder:

trait ObjectEncoder[-A] extends Encoder[A] {
 def encode(value: A): Json.Obj
 def zip[B](that: ObjectEncoder[B]): ObjectEncoder[(A, B)] = ...
}

І навпаки, декодування значення типу A полягає у перетворенні значення типу Json у значення типу A:

trait Decoder[+A] {
 /**
   * @param data The data to de-serialize
   * @return The decoded value wrapped in `Some`, or `None` if decoding failed
   */
 def decode(data: Json): Option[A]
}

Зверніть увагу, що операція декодування повертає Option[A] замість просто A, оскільки процес декодування може завершитись невдачею(None означає, що було неможливо створити A з наданих даних JSON).
Неявні (Implicit) кодери та декодери визначаються відповідно в трейтах EncoderInstances та DecoderInstances. Ці трейти успадковуються компаньйон об'єктами Encoder та Decoder відповідно. Завдяки цьому ми робимо неявні екземпляри автоматично доступними, усуваючи необхідність їх явного імпорту.
Вам надається функція parseJson, яка парсить blob-файл JSON, представленого у вигляді  String, у значення Json, і функція renderJson, яка перетворює значення Json BLOB-об'єкту у String. Ви можете використовувати їх для перетворень з кодерами та декодерами, які ви пишете. Функції parseJson та renderJson визначені у файлі Util.scala.

Ваша задача

Ваша робота полягає у написанні екземплярів кодерів, які можна об’єднати для створення екземплярів кодерів для більш складних типів. Ви почнете з написання екземплярів для базових типів (таких як String або Boolean), а в кінцевому підсумку ви напишете екземпляри для case класів.
Відкрийте файл codecs.scala. Він містить визначення типів Json, Encoder, ObjectEncoder і Decoder. Завершіть частково реалізовані неявні визначення (замініть ??? на належні реалізації) та введіть нові неявні визначення за необхідності (шукайте коментарі TODO).
У будь-який час ви можете стежити за своїм прогресом, запустивши тестове завдання sbt. Ви також можете використовувати завдання run sbt для запуску основної програми, визначеної внизу файлу codecs.scala. Нарешті, ви можете створити сеанс REPL із завданням консолі sbt, а потім експериментувати зі своїм кодом:
bt:progfun2-codecs> console

scala> import codecs._ import codecs._

scala> Util.parseJson(""" { "name": "Bob", "age": 10 } """)
val res0: Option[codecs.Json] = Some(Obj(Map(name -> Str(Bob), age -> Num(10))))

scala> res0.flatMap(_.decodeAs[Person]) // This will crash until you implement it in this assignment
val res1: Option[codecs.Person] = Some(Person(Bob,10))

scala> implicitly[Encoder[Int]]
val res2: codecs.Encoder[Int] = codecs.Encoder$$anon$1@74d8fde0

scala> res2.encode(42)
val res3: codecs.Json = Num(42)

scala> :quit 

Пам'ятайте, що якщо ви внесете будь-які зміни в код, вам потрібно буде вийти з консолі та запустити її знову, щоб запустити оновлений код.
Основні кодери
Почніть із реалізації неявних екземплярів Encoder [String] і Encoder [Boolean] та відповідних неявних екземплярів Decoder [Int], Decoder [String] і Decoder [Boolean].
Підказка: використовуйте надані методи Encoder.fromFunction, Decoder.fromFunction та Decoder.fromPartialFunction.
Переконайтеся, що декодер Int не приймає числа з плаваючою комою JSON. Для реалізації зверніться до функцій з BigDecimal

Похідні кодери

Наступний крок полягає у реалізації кодера для списків елементів типу A, з використанням реалізованого кодеру для типу A. Екземпляр кодера для списків вже реалізований. Заповніть визначення відповідного декодера.
Після того, як ви визначили кодер та декодер для списків, ви можете викликати їх для будь-якого списку, що містить елементи, які можна кодувати та декодувати. Ви можете спробувати, наприклад, оцінити такі вирази в REPL:

implicitly[Encoder[List[Int]]]
implicitly[Decoder[List[Boolean]]]

кодери об’єктів JSON

Далі реалізуйте кодери для об’єктів JSON. Підхід полягає у визначенні кодерів для об'єктів JSON, що мають одне поле, а потім комбінуванні таких кодерів для обробки об'єктів JSON з кількома полями.
Наприклад, розглянемо такий об'єкт JSON з одним полем x:
{
  "x": 1
}
Кодер для цього об'єкта JSON можна визначити, використовуючи надану операцію ObjectEncoder.field, яка приймає ім'я поля як параметр і неявний кодер для значення поля:
val xField = ObjectEncoder.field[Int]("x")

Ось приклад використання кодера xField:

scala> xField.encode(42)
val res0: codecs.Json.Obj = Obj(Map(x -> Num(42)))

Нарешті, щоб визначити кодер, що створює об’єкт JSON з кількома полями, ви можете об’єднати два екземпляри ObjectEncoder із операцією zip. Ця операція повертає ObjectEncoder, що створює об'єкт JSON із полями двох комбінованих кодерів. Наприклад, ви можете визначити кодер, що створює об'єкт з двома полями x і y, наступним чином:
val pointEncoder: ObjectEncoder[(Int, Int)] = {
 val xField = ObjectEncoder.field[Int]("x")
 val yField = ObjectEncoder.field[Int]("y")
 xField.zip(yField)
}

Реалізуйте операцію Decoder.field, що відповідає операції ObjectEncoder.field.
Кодери для Case  класів
Операція zip, згадана в попередньому розділі, повертає лише кодери для кортежів. Було б зручніше працювати з типами даних високого рівня (наприклад, тип даних Point, а не пара координат).
І кодери, і декодери мають операцію transform , яка може бути використана для перетворення значень типу Json, кодованих або декодованих відповідно. Ви можете побачити в завданні, як він використовується для визначення даного Encoder[Person].
Визначте відповідний Decoder[Person], а потім визначте неявний кодер  implicit Encoder[Contacts]  та implicit Decoder[Contacts].

Індивідуальні завдання 

Реалізуйте кодування та декодування для класу case class Password(value: String, code: List[Int])
