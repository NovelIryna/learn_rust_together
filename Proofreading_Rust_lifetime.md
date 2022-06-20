**Поширені помилкові уявлення про життєві цикли у Rust** 

Original: [https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md#4-my-code-isn't-generic-and-does not-have-lifetimes](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md#4-my-code-isnt-generic-and-doesnt-have-lifetimes) 

*19 травня 2020 ·#rust · #lifetimes*

Зміст




- [Вступ](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.ghlf17y5ason)
- [Помилкові уявлення](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.jxfakzvgj215)
  - [1) T містить лише власні типи](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.uy3oq6hovyz5)
  - [2) якщо T: 'static, тоді T має бути дійсним для всієї програми](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.ogcy37f29tfb)
  - [3) &'a T і  T: 'a - це одне й те ж](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.vxc6u9ur3is6)
  - [4) мій код не є узагальненим і не має  ](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.snx9u134awrj)тривалості життя
  - [5) якщо він компілюється, мої лайфтайм анотації правильні](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.a60je43wspsv)
  - [6)  об'єкти запакованих трейтів (boxed traits) не мають тривалості життя](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.s7ind4f5fo7)
  - [7) повідомлення про помилки компілятора підкажуть мені,  які є ](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.kf960abd7y8)способи виправлення помилок програми
  - [8) тривалість життя може зростати і скорочуватися під час виконання](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.e40dtmmnjomg)
  - [9) переведення mut refs у  shared  є безпечним](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.79b8488lx0u1)
  - [10) замикання дотримуються тих самих правил виключення лайфтайм-параметрів, що й функції](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.8ri78ivxwsx8)
- [Висновки](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.6jker1t0dpvk)
- [Корисні посилання](https://docs.google.com/document/d/1Qt-lpSVvdysVvhtmFJSe6juAGVD14zyhuIam5IGxdkw/edit#heading=h.hxe57941x73j)



**Вступ**

Колись я стикався з багатьма  хибними твердженнями, і я бачу, що сьогодні багато новачків борються з цими помилками. Деякі з моїх термінів можуть бути нестандартними, тому ось таблиця скорочених фраз, які я використовую, і що на мою думку мають означати.



|**Фраза**|**Скорочення для**|
| :-: | :-: |
|T|<p>1) набір, що містить усі можливі типи або</p><p>2) якийсь тип у цьому наборі</p>|
|owned тип|деякий не довідковий тип, наприклад i32, String, Vec, тощо|
|<p>1) borrowed тип або</p><p>2) тип ref</p>|деякий еталонний або довідковий тип незалежно від властивості здатності бути зміненим чи ні , наприклад &i32, &mut i32, і т.д|
|<p>1) mut ref або</p><p>2) exclusive ref</p>|ексклюзивне змінне посилання, тобто &mut T|
|<p>1) immut ref *or*</p><p>2) shared ref</p>|спільне(shared) незмінне посилання, тобто &T|
**Помилкові уявлення**

Коротше кажучи: час життя змінної (лайфтайм) - це те, як довго дані, на які вона вказує, можуть бути статично перевірені компілятором на те, що вони дійсні за поточною адресою пам'яті. А зараз я приділю наступні ~6500 слів, щоб докладніше розповісти про те, де люди зазвичай плутаються.

**1) T містить лише  власні типи**

Це хибне уявлення більше стосується генериків, ніж тривалості життя, але генерики та терміни життя тісно переплетені в Rust, тому неможливо говорити про одне, не говорячи також про інше. У будь-якому випадку:

Коли я вперше почав вивчати Rust, я зрозумів, що i32, &i32, і &mut i32 є різними типами. Я також зрозумів, що деяка змінна загального типу T представляє набір, який містить усі можливі типи. Проте, незважаючи на розуміння обох цих речей окремо, я не зміг зрозуміти їх разом. У моєму новачковому розумі Rust я думав, що генерики працюють так:

|**Тип Змінної**|T|&T|&mut T|
| :- | :- | :- | :- |
|**Приклади**|i32|&i32|&mut i32|


T містить усі owned типи. &T містить усі незмінно запозичені типи. &mut T містить усі змінно запозичені типи. T, &T, і &mut T — несполучні скінченні множини . Гарно, просто, чисто, легко, інтуїтивно зрозуміло і абсолютно неправильно. Ось як генерики насправді працюють у Rust:

|**Тип Змінної**|T|&T|&mut T|
| :- | :- | :- | :- |
|**Приклади**|i32, &i32, &mut i32, &&i32, &mut &mut i32, ...|&i32, &&i32, &&mut i32, ...|&mut i32,       &mut &mut i32, &mut &i32, ...|


T, &T, і &mut T - це все нескінченні множини, оскільки можна запозичити нескінченний тип ad-infinitum. T є надмножиною обох &T і &mut T. &T і &mut T - множини, що не перетинаються. Ось кілька прикладів, які підтверджують ці концепції:

trait Trait {}

impl<T> Trait for T {}

impl<T> Trait for &T {} // ❌

impl<T> Trait for &mut T {} // ❌

Наведена вище програма компілюється не так, як очікувалося:

error[E0119]: conflicting implementations of trait `Trait` for type `&\_`:

` `--> src/lib.rs:5:1

`  `|

3 | impl<T> Trait for T {}

`  `| ------------------- first implementation here

4 |

5 | impl<T> Trait for &T {}

`  `| ^^^^^^^^^^^^^^^^^^^^ conflicting implementation for `&\_`

error[E0119]: conflicting implementations of trait `Trait` for type `&mut \_`:

` `--> src/lib.rs:7:1

`  `|

3 | impl<T> Trait for T {}

`  `| ------------------- first implementation here

...

7 | impl<T> Trait for &mut T {}

`  `| ^^^^^^^^^^^^^^^^^^^^^^^^ conflicting implementation for `&mut \_`



Компілятор не дозволяє нам визначити реалізацію Trait для &T, і &mut T , оскільки це буде конфліктувати з реалізацією Trait для T, яка вже включає всі &T і &mut T. Програма, наведена нижче, компілюється, як очікувалося, оскільки &T і &mut T не перетинаються:

trait Trait {}

impl<T> Trait for &T {} // ✅

impl<T> Trait for &mut T {} // ✅

Ключові тези:

- T є надмножиною і &T і &mut T
- &T і &mut T є множинами, що не перетинаються

**2) якщо T: 'static, тоді T має бути дійсним для всієї програми**

Наслідки помилкового  твердження:

- T: 'static слід читати як *" T має 'static лайфтайм"*
- &'static T і T: 'static - це одне й те саме
- якщо T: 'static, тоді T має бути незмінним
- якщо T: 'static, тоді T може бути створений лише під час компіляції

Більшість Rust початківців  вперше знайомляться з 'static лайфтаймом у прикладі коду, який виглядає приблизно так:

fn main() {

`    `let str\_literal: &'static str = "str literal";

}

Їм кажуть, що "str literal"  жорстко закодований у скомпільований двійковий файл і завантажується в пам’ять лише для читання під час виконання, тому він незмінний і дійсний для всієї програми, і саме це робить її “static”. Ці поняття додатково підкріплюються правилами щодо визначення static змінних за допомогою ключового слова static .

// Примітка: Цей приклад виключно з метою демонстрації.

// Ніколи не використовуй `static mut`.Це можна сказати, постріл собі в 

// ногу. Є безпечні шаблони для глобальних змінюваних синглтонів у Rust, 

// але вони виходять за рамки цієї статті

static BYTES: [u8; 3] = [1, 2, 3];

static mut MUT\_BYTES: [u8; 3] = [1, 2, 3];

fn main() {

`   `MUT\_BYTES[0] = 99; // ❌ - робити static змінним не є безпечним 

`    `unsafe {

`        `MUT\_BYTES[0] = 99;

`        `assert\_eq!(99, MUT\_BYTES[0]);

`    `}

}

Щодо static змінних

- їх можна створити лише під час компіляції
- вони повинні бути незмінними, намагання зробити їх змінними не є безпечним
- вони є дійсними для всієї програми

'static лайфтайм, ймовірно, був названий на честь тривалості життя static змінних за замовчуванням, чи не так? Тож це має сенс, що 'static лайфтайм має відповідати тим самим правилам, чи не так?

Так, але тип *з* 'static  лайфтаймом відрізняється від типу *, обмеженого* лайфтаймом* 'static. Останній можна динамічно розподіляти під час виконання, можна безпечно та вільно робити змінним, відкидати та може жити протягом довільної тривалості. 

На цьому етапі важливо відрізняти &'static T від T: 'static.

&'static T є незмінним посиланням на деякі T, які можна безпечно зберігати необмежено довго, в тому числі до кінця програми. Це можливо лише в тому випадку, якщо T саме по собі  є незмінним і не переміщується *після створення посилання* . T не потрібно створювати під час компіляції. Можна генерувати випадково динамічно розподілені дані під час виконання та повертати 'static посилання на них ціною витоку пам’яті, наприклад

use rand;

// генеруємо рандомний 'static str refs під час виконання

fn rand\_str\_generator() -> &'static str {

`    `let rand\_string = rand::random::<u64>().to\_string();

`    `Box::leak(rand\_string.into\_boxed\_str())

}

T: 'static - це деякі T, які можна безпечно утримувати необмежено довго, в тому числі до кінця програми. T: 'static включає всі &'static T, однак він також включає всі належні типи, наприклад String, Vec, тощо. Власник деяких даних гарантує, що дані ніколи не будуть визнані недійсними, доки власник їх зберігає, тому власник може безпечно зберігати дані необмежено довго, в т.ч. до кінця програми. T: 'static слід читати як *« T обмежений* лайфтаймом *'static »* , а не *« T має лайфтайм'static»* . Програма, яка допоможе проілюструвати ці поняття:

use rand;

fn drop\_static<T: 'static>(t: T) {

`    `std::mem::drop(t);

}

fn main() {

`    `let mut strings: Vec<String> = Vec::new();

`    `for \_ in 0..10 {

`        `if rand::random() {

`            `// усі рядки(strings)  рандомно згенеровані 

`            `// і динамічно розподіляються під час виконання

`            `let string = rand::random::<u64>().to\_string();

`            `strings.push(string);

`        `}

`    `}

`    `// рядки (strings) є owned типами , тому вони обмежені 'static -ом

`    `for mut string in strings {

`        `// усі рядки (strings) є змінними

`        `string.push\_str("a mutation");

`        `// усі рядки (strings)є такими, що можна відкинути

`        `drop\_static(string); // ✅

`    `}

`    `// усі рядки(strings) були оголошені недійсними до завершення програми

`     `println!("I am the end of the program");

}

Ключові тези:

- T: 'static слід читати як *" T обмежений 'static* часом життя програми *"*
- якщо T: 'static , то T може бути запозиченим типом з 'static лайфтаймом *або* owned типом
- оскільки T: 'static включає власні типи, це означає, що T
  - можна динамічно розподіляти під час виконання
  - не має бути дійсним для всієї програми
  - може бути безпечно і вільно змінюваним 
  - може динамічно скидатися під час виконання
  - може мати різну тривалість життя

**3) &'a T і  T: 'a - це одне й те ж**

Ця помилка є узагальненою версією наведеної вище.

&'a T вимагає та має на увазі T: 'a, оскільки посилання на T лайфтайм  'a не може бути дійсним для 'a, якщо T саме по собі не є дійсним для 'a. Наприклад, компілятор Rust ніколи не дозволить конструкцію типу         &'static Ref<'a, T> , тому що якщо Ref дійсний тільки для 'a , ми не можемо зробити 'static посилання на нього.

T: 'a включає усі &'a T, але зворотне не вірно.

// приймає тільки ref типи обмежені 'a

fn t\_ref<'a, T: 'a>(t: &'a T) {}

// приймає  будь-які типи обмежені 'a

fn t\_bound<'a, T: 'a>(t: T) {}

// owned типи,які включають посилання 

struct Ref<'a, T: 'a>(&'a T);

fn main() {

`    `let string = String::from("string");

`    `t\_bound(&string); // ✅

`    `t\_bound(Ref(&string)); // ✅

`    `t\_bound(&Ref(&string)); // ✅

`    `t\_ref(&string); // ✅

`    `t\_ref(Ref(&string)); // ❌ - очікувано реф тип,а знайдено структуру 

`    `t\_ref(&Ref(&string)); // ✅

`    `// рядок(стрінг) обмежений 'static , який обмежений 'a

`    `t\_bound(string); // ✅

}

Ключові тези:

- T: 'a є більш загальним і більш гнучким, ніж &'a T
- T: 'a приймає власні типи, власні типи, які містять посилання, і посилання
- &'a T приймає лише посилання
- якщо T: 'static тоді , T: 'a оскільки 'static >= 'a для всіх 'a

**4) мій код не є узагальненим і не має тривалостей життя** 

Помилкові твердження

- можна уникнути використання генериків і лайфтаймів

Це втішне помилкове уявлення зберігається завдяки правилам лайфтайм-параметрів Rust, які дозволяють вам пропускати анотації за весь період у функціях, оскільки засіб перевірки Rust запозичує їх за цими правилами:

- кожне вхідне посилання на функцію отримує певний час життя 
- якщо є рівно один вхідний лайфтайм, він застосовується до всіх вихідних посилань
- якщо є кілька вхідних лайфтаймів , але один з них є &self або &mut self ,тоді лайфтайм self застосовується до всіх вихідних посилань
- інакше терміни життя вихідних даних мають бути явними

Це дуже багато , тому давайте розглянемо кілька прикладів:

// виключено

fn print(s: &str);

// розширено

fn print<'a>(s: &'a str);

// виключено

fn trim(s: &str) -> &str;

// розширено

fn trim<'a>(s: &'a str) -> &'a str;

// неправильно, ми не можемо визначити вихідний лайфтайм, немає вхідного

fn get\_str() -> &str;

// включення явних опцій

fn get\_str<'a>() -> &'a str; // узагальнена версія

fn get\_str() -> &'static str; // 'static версія

// неправильно, ми не можемо визначити вихідний лайфтайм,багато вхідних

fn overlap(s: &str, t: &str) -> &str;

// включення  явних (але частково виключених) опцій

fn overlap<'a>(s: &'a str, t: &str) -> &'a str; // output can't outlive s

fn overlap<'a>(s: &str, t: &'a str) -> &'a str; // output can't outlive t

fn overlap<'a>(s: &'a str, t: &'a str) -> &'a str; // output can't outlive s & t

fn overlap(s: &str, t: &str) -> &'static str; // output can outlive s & t

fn overlap<'a>(s: &str, t: &str) -> &'a str; // no relationship between input & output lifetimes

// розширено

fn overlap<'a, 'b>(s: &'a str, t: &'b str) -> &'a str;

fn overlap<'a, 'b>(s: &'a str, t: &'b str) -> &'b str;

fn overlap<'a>(s: &'a str, t: &'a str) -> &'a str;

fn overlap<'a, 'b>(s: &'a str, t: &'b str) -> &'static str;

fn overlap<'a, 'b, 'c>(s: &'a str, t: &'b str) -> &'c str;

// виключено

fn compare(&self, s: &str) -> &str;

// розширено

fn compare<'a, 'b>(&'a self, &'b str) -> &'a str;

Якщо ви коли-небудь писали

- структурний метод 
- функцію, яка приймає посилання
- функцію, яка повертає посилання
- загальну функцію
- трейт об’єкт (докладніше про це пізніше)
- замикання (докладніше про це пізніше)

тоді ваш код має узагальнені  приховані лайфтайм анотації.

Ключові тези

- майже весь код Rust є узагальненим, і всюди є приховані лайфтайм анотації

**5) якщо він компілюється, мої лайфтайм анотації правильні**

Помилкові твердження

- Правила виключення лайфтайм-параметрів Rust для функцій завжди правильні
- Перевірка запозичень від Rust завжди правильна, технічно *та семантично*
- Rust знає про семантику моєї програми більше, ніж я

Програма Rust може бути технічно компілюваною, але все одно семантично неправильною. Візьмемо для прикладу це:

struct ByteIter<'a> {

`    `remainder: &'a [u8]

}

impl<'a> ByteIter<'a> {

`    `fn next(&mut self) -> Option<&u8> {

`        `if self.remainder.is\_empty() {

`            `None

`        `} else {

`            `let byte = &self.remainder[0];

`            `self.remainder = &self.remainder[1..];

`            `Some(byte)

`        `}

`    `}

}

fn main() {

`    `let mut bytes = ByteIter { remainder: b"1" };

`    `assert\_eq!(Some(&b'1'), bytes.next());

`    `assert\_eq!(None, bytes.next());

}

ByteIter є ітератором, який перебирає фрагмент байтів. Ми пропускаємо реалізацію Iterator трейту для стислості. Здається,що це працює добре, але що робити, якщо ми хочемо перевірити пару байтів за раз?

fn main() {

`    `let mut bytes = ByteIter { remainder: b"1123" };

`    `let byte\_1 = bytes.next();

`    `let byte\_2 = bytes.next();

`    `if byte\_1 == byte\_2 { // ❌

`        `// роби щось

`    `}

}

Ой-ой! Помилка компіляції:

error[E0499]: cannot borrow `bytes` as mutable more than once at a time

`  `--> src/main.rs:20:18

`   `|

19 |     let byte\_1 = bytes.next();

`   `|                  ----- first mutable borrow occurs here

20 |     let byte\_2 = bytes.next();

`   `|                  ^^^^^ second mutable borrow occurs here

21 |     if byte\_1 == byte\_2 {

`   `|        ------ first borrow later used here



Я думаю, ми можемо скопіювати кожен байт. Копіювання - це нормально, коли ми працюємо з байтами, але якщо ми перетворили ByteIter на узагальнений ітератор фрагментів, який може виконувати ітерацію по будь-якому &'a [T], ми можемо використовувати його в майбутньому з типами, які можуть бути дуже дорогими або неможливими для копіювання та клонування. Ну що ж, я думаю, ми нічого не можемо з цим зробити, код компілюється, тому анотації протягом життя повинні бути правильними, чи не так?

Ні, фактично джерелом помилки є поточні життєві анотації! Це особливо важко помітити, оскільки лайфтайм анотації роботи з помилками вилучені. Давайте розширимо виключені лайфтайми, щоб ясніше поглянути на проблему:

struct ByteIter<'a> {

`    `remainder: &'a [u8]

}

impl<'a> ByteIter<'a> {

`    `fn next<'b>(&'b mut self) -> Option<&'b u8> {

`        `if self.remainder.is\_empty() {

`            `None

`        `} else {

`            `let byte = &self.remainder[0];

`            `self.remainder = &self.remainder[1..];

`            `Some(byte)

`        `}

`    `}

}

Це зовсім не допомогло. Я все ще розгублений. Ось гарний лайфхак, про який знають лише професіонали Rust: дайте своїм лайфтайм анотаціям описові назви. Давайте спробуємо ще раз:

struct ByteIter<'remainder> {

`    `remainder: &'remainder [u8]

}

impl<'remainder> ByteIter<'remainder> {

`    `fn next<'mut\_self>(&'mut\_self mut self) -> Option<&'mut\_self u8> {

`        `if self.remainder.is\_empty() {

`            `None

`        `} else {

`            `let byte = &self.remainder[0];

`            `self.remainder = &self.remainder[1..];

`            `Some(byte)

`        `}

`    `}

}

Кожен повернутий байт анотується з 'mut\_self ,але байти явно походять з 'remainder ! Давайте виправимо це.

struct ByteIter<'remainder> {

`    `remainder: &'remainder [u8]

}

impl<'remainder> ByteIter<'remainder> {

`    `fn next(&mut self) -> Option<&'remainder u8> {

`        `if self.remainder.is\_empty() {

`            `None

`        `} else {

`            `let byte = &self.remainder[0];

`            `self.remainder = &self.remainder[1..];

`            `Some(byte)

`        `}

`    `}

}

fn main() {

`    `let mut bytes = ByteIter { remainder: b"1123" };

`    `let byte\_1 = bytes.next();

`    `let byte\_2 = bytes.next();

`    `std::mem::drop(bytes); // зараз ми можемо навіть викинути(дропнути) ітератор

`    `if byte\_1 == byte\_2 { // ✅

`        `// роби щось

`    `}

}

Тепер, коли ми озираємося на попередню версію нашої програми, вона, очевидно, була неправильною, то чому Rust її скомпілював? Відповідь проста: це було безпечно для пам’яті.

Засіб перевірки запозичень Rust дбає лише про анотації протягом життя в програмі в тій мірі, в якій він може використовувати їх для статичної перевірки безпеки пам’яті програми. Rust із задоволенням компілює програми, навіть якщо лайфтайм анотації мають семантичні помилки, і наслідком цього є те, що програма стає невиправдано обмеженою.

Ось короткий приклад, протилежний попередньому: правила виключення за весь період життя Rust у цьому випадку є семантично правильними, проте ми ненавмисно пишемо дуже обмежуючий метод із нашими власними не обов’язковими явними лайфтайм анотаціями.

#[derive(Debug)]

struct NumRef<'a>(&'a i32);

impl<'a> NumRef<'a> {

`    `// моя структура є узагальненою над ‘a, тому це означає, що мені потрібно додати анотації

`    `// мої self параметри з 'a також, правильно? (відповідь: ні,це не так)

`    `fn some\_method(&'a mut self) {}

}

fn main() {

`    `let mut num\_ref = NumRef(&5);

`    `num\_ref.some\_method(); // змінно запозичує num\_ref до кінця свого життя

`    `num\_ref.some\_method(); // ❌

`    `println!("{:?}", num\_ref); // ❌

}

Якщо у нас є узагальнена структура 'a, ми майже ніколи не хочемо писати метод з &'a mut self приймачем. Те, що ми повідомляємо Rust, так це *«цей метод змінно запозичуватиме структуру протягом усього терміну її існування»* . На практиці це означає, що засіб перевірки запозичень у Rust дозволить щонайбільше один виклик some\_method, перш ніж структура стане постійно змінно запозиченою і, таким чином, непридатною для використання. Випадки використання для цього надзвичайно рідкісні, але наведений вище код дуже легко написати для розгублених новачків, і він компілюється. Виправлення полягає в тому, щоб не додавати непотрібні явні лайфтайм анотації і дозволити правилам лайфтайм-параметрів Rust обробляти це:

#[derive(Debug)]

struct NumRef<'a>(&'a i32);

impl<'a> NumRef<'a> {

`    `// ніяких більше 'a на mut self

`    `fn some\_method(&mut self) {}

`    `// рядок вище розгортається у

`    `fn some\_method\_desugared<'b>(&'b mut self){}

}

fn main() {

`    `let mut num\_ref = NumRef(&5);

`    `num\_ref.some\_method();

`    `num\_ref.some\_method(); // ✅

`    `println!("{:?}", num\_ref); // ✅

}

Ключові тези:

- Правила виключення лайфтайм-параметрів Rust для функцій не завжди підходять для кожної ситуації
- Rust не знає більше про семантику вашої програми, ніж ви
- дайте своїм лайфтайм анотаціям описові назви
- намагайтеся пам’ятати про те, де ви розміщуєте явні лайфтайм анотації  і чому





**6)  об'єкти запакованих трейтів (boxed traits) не мають тривалості життя**  
**


*Раніше ми обговорювали правила вилучення* лайфтайм-параметрів Rust *для функцій* . Rust також має правила виключення лайфтаймів для трейт об’єктів, а саме:

- якщо трейт об’єкти використовується як аргумент типу для узагальненого типу, то його межа життя виводиться з вмістимого (containing) типу
  - якщо є унікальне зв'язування з вмістом, то воно використовується
  - якщо є більше одного зв'язку з вмістимого типу, то має бути вказано явне обмеження
- якщо не стосується наведеного вище , тоді
  - якщо трейт визначений за допомогою однієї межі життя, то використовується ця межа
  - якщо 'static використовується для будь-якого лайфтайму, то 'static використовується
  - якщо трейт не має обмежень по тривалості життя, то її тривалість визначається у виразах і є 'static за межами виразів

Все це звучить дуже складно, але може бути просто узагальнено як *«межі тривалості життя трейт об’єкта виводиться з контексту».* Розглянувши кілька прикладів, ми побачимо, що висновки, пов’язані з життям, досить інтуїтивно зрозумілі, тому нам не потрібно запам’ятовувати формальні правила:

use std::cell::Ref;

trait Trait {}

// виключено

type T1 = Box<dyn Trait>;

// розширено, Box<T> не має лайфтайм обмежень на T, тож виводиться як 'static

type T2 = Box<dyn Trait + 'static>;

// виключено

impl dyn Trait {}

// розширено

impl dyn Trait + 'static {}

// виключено

type T3<'a> = &'a dyn Trait;

// розширено, &'a T вимагає T: 'a,тому виводиться як 'a

type T4<'a> = &'a (dyn Trait + 'a);

// виключено

type T5<'a> = Ref<'a, dyn Trait>;

// eрозширено, Ref<'a, T> вимагає T: 'a,тому виводиться як 'a

type T6<'a> = Ref<'a, dyn Trait + 'a>;

trait GenericTrait<'a>: 'a {}

// виключено

type T7<'a> = Box<dyn GenericTrait<'a>>;

// розширено

type T8<'a> = Box<dyn GenericTrait<'a> + 'a>;

// виключено

impl<'a> dyn GenericTrait<'a> {}

// розширено

impl<'a> dyn GenericTrait<'a> + 'a {}

Конкретні типи, які реалізують трейти, можуть мати посилання, і, таким чином, вони також мають межі часу життя, а отже, їхні відповідні об’єкти ознак мають межі часу життя. Також ви можете реалізувати трейти безпосередньо для посилань, які, очевидно, мають межі тривалості життя:

trait Trait {}

struct Struct {}

struct Ref<'a, T>(&'a T);

impl Trait for Struct {}

impl Trait for &Struct {} // impl Трейт безпосередньо на ref type

impl<'a, T> Trait for Ref<'a, T> {} // impl Трейт на тип, що містить refs

У будь-якому випадку, це варто розглянути, оскільки це часто збиває новачків з пантелику, коли вони переробляють функцію з використання трейт об’єктів до загальних або навпаки. Візьмемо для прикладу цю програму:

use std::fmt::Display;

fn dynamic\_thread\_print(t: Box<dyn Display + Send>) {

`    `std::thread::spawn(move || {

`        `println!("{}", t);

`    `}).join();

}

fn static\_thread\_print<T: Display + Send>(t: T) { // ❌

`    `std::thread::spawn(move || {

`        `println!("{}", t);

`    `}).join();

}

Він видає цю помилку компіляції:

error[E0310]: the parameter type `T` may not live long enough

`  `--> src/lib.rs:10:5

`   `|

9  | fn static\_thread\_print<T: Display + Send>(t: T) {

`   `|                        -- help: consider adding an explicit lifetime bound...: `T: 'static +`

10 |     std::thread::spawn(move || {

`   `|     ^^^^^^^^^^^^^^^^^^

`   `|

note: ...so that the type `[closure@src/lib.rs:10:24: 12:6 t:T]` will meet its required lifetime bounds

`  `--> src/lib.rs:10:5

`   `|

10 |     std::thread::spawn(move || {

`   `|     ^^^^^^^^^^^^^^^^^^



Добре, компілятор розповідає нам, як вирішити проблему, тож давайте вирішимо проблему.

use std::fmt::Display;

fn dynamic\_thread\_print(t: Box<dyn Display + Send>) {

`    `std::thread::spawn(move || {

`        `println!("{}", t);

`    `}).join();

}

fn static\_thread\_print<T: Display + Send + 'static>(t: T) { // ✅

`    `std::thread::spawn(move || {

`        `println!("{}", t);

`    `}).join();

}

Зараз він компілюється, але ці дві функції виглядають незручно одна біля одної, чому друга функція вимагає 'static межі на T ,де перша функція ні? Це підступне питання. Використовуючи правила виключення за весь час, Rust автоматично визначає 'static межу в першій функції, тому фактично вони обидві мають 'static межі. Ось що бачить компілятор Rust:

use std::fmt::Display;

fn dynamic\_thread\_print(t: Box<dyn Display + Send + 'static>) {

`    `std::thread::spawn(move || {

`        `println!("{}", t);

`    `}).join();

}

fn static\_thread\_print<T: Display + Send + 'static>(t: T) {

`    `std::thread::spawn(move || {

`        `println!("{}", t);

`    `}).join();

}

Ключові тези:

- всі трейт об'єкти мають певні межі тривалості життя за замовчуванням

**7) повідомлення про помилки компілятора підкажуть мені, які є способи виправлення помилок програми** 

Помилкові твердження

- Правила виключення Rust для трейт об’єктів завжди правильні
- Rust знає про семантику моєї програми більше, ніж я

Це помилкове уявлення є двома попередніми помилками, об’єднаними в один приклад:

use std::fmt::Display;

fn box\_displayable<T: Display>(t: T) -> Box<dyn Display> { // ❌

`    `Box::new(t)

}

Видає цю помилку:

error[E0310]: the parameter type `T` may not live long enough

` `--> src/lib.rs:4:5

`  `|

3 | fn box\_displayable<T: Display>(t: T) -> Box<dyn Display> {

`  `|                    -- help: consider adding an explicit lifetime bound...: `T: 'static +`

4 |     Box::new(t)

`  `|     ^^^^^^^^^^^

`  `|

note: ...so that the type `T` will meet its required lifetime bounds

` `--> src/lib.rs:4:5

`  `|

4 |     Box::new(t)

`  `|     ^^^^^^^^^^^



Гаразд, давайте виправимо це так, як компілятор каже нам виправити це, не звертаючи уваги на той факт, що він автоматично визначає 'static лайфтайм для нашого запакованого трейт об’єкта, не повідомляючи нам, і його рекомендоване виправлення засноване на цьому невиявленому факті:

use std::fmt::Display;

fn box\_displayable<T: Display + 'static>(t: T) -> Box<dyn Display> { // ✅

`    `Box::new(t)

}

Отже, програма компілюється зараз... але чи це те, чого ми насправді хочемо? Можливо, але, може й ні. Компілятор не згадав жодних інших виправлень, але це також було б доречним:

use std::fmt::Display;

fn box\_displayable<'a, T: Display + 'a>(t: T) -> Box<dyn Display + 'a> { // ✅

`    `Box::new(t)

}

Ця функція приймає всі ті ж аргументи, що й попередня версія, а також багато іншого! Чи це робить його краще? Не обов’язково, це залежить від вимог та обмежень нашої програми. Цей приклад трохи абстрактний, тому давайте подивимося на більш простий і очевидний випадок:

fn return\_first(a: &str, b: &str) -> &str { // ❌

`    `a

}

Викидає:

error[E0106]: missing lifetime specifier

` `--> src/lib.rs:1:38

`  `|

1 | fn return\_first(a: &str, b: &str) -> &str {

`  `|                    ----     ----     ^ expected named lifetime parameter

`  `|

`  `= help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `a` or `b`

help: consider introducing a named lifetime parameter

`  `|

1 | fn return\_first<'a>(a: &'a str, b: &'a str) -> &'a str {

`  `|                ^^^^    ^^^^^^^     ^^^^^^^     ^^^



Повідомлення про помилку рекомендує анотувати обидва вхідні та вихідні дані з однаковим терміном життя. Якщо ми це зробимо, наша програма буде компілюватися, але ця функція буде надмірно обмежувати тип повернення. Насправді ми хочемо цього:

fn return\_first<'a>(a: &'a str, b: &str) -> &'a str { // ✅

`    `a

}

Ключові тези:

- Правила виключення лайфтайм-параметрів Rust для трейт об’єктів не завжди підходять для кожної ситуації
- Rust не знає більше про семантику вашої програми, ніж ви
- Повідомлення про помилки компілятора Rust пропонують виправлення, які змусять скомпілювати вашу програму, що не те саме, що виправлення, які змусять вас скомпілювати програму *та* найкраще відповідати вимогам вашої програми

**8) тривалість життя може зростати і скорочуватися під час виконання**

Наслідки помилкового уявлення

- Типи контейнерів можуть змінювати посилання під час виконання, щоб змінити їхні лайфтайми
- Програма перевірки запозичень Rust  виконує розширений аналіз потоку керування

Це не компілюється:

struct Has<'lifetime> {

`    `lifetime: &'lifetime str,

}

fn main() {

`    `let long = String::from("long");

`    `let mut has = Has { lifetime: &long };

`    `assert\_eq!(has.lifetime, "long");

`    `{

`        `let short = String::from("short");

`        `// "змінює" на короткий лайфтайм

`        `has.lifetime = &short;

`        `assert\_eq!(has.lifetime, "short");

`        `// "змінює назад" на довгий лайфтайм (але не зовсім так)

`        `has.lifetime = &long;

`        `assert\_eq!(has.lifetime, "long");

`        `// `короткий` викинувся тут

`    `}

`    `assert\_eq!(has.lifetime, "long"); // ❌ - `короткий` все ще "borrowed" після викидання

}

Видає:

error[E0597]: `short` does not live long enough

`  `--> src/main.rs:11:24

`   `|

11 |         has.lifetime = &short;

`   `|                        ^^^^^^ borrowed value does not live long enough

...

15 |     }

`   `|     - `short` dropped here while still borrowed

16 |     assert\_eq!(has.lifetime, "long");

`   `|     --------------------------------- borrow later used here



Це також не компілюється, видає ту саму помилку, що й вище:

struct Has<'lifetime> {

`    `lifetime: &'lifetime str,

}

fn main() {

`    `let long = String::from("long");

`    `let mut has = Has { lifetime: &long };

`    `assert\_eq!(has.lifetime, "long");

`    `// цей блок ніколи не запрацює

`    `if false {

`        `let short = String::from("short");

`        `// "змінює" на короткий лайфтайм

`        `has.lifetime = &short;

`        `assert\_eq!(has.lifetime, "short");

`        `// "змінює назад" на довгий лайфтайм (але не зовсім так)

`        `has.lifetime = &long;

`        `assert\_eq!(has.lifetime, "long");

`        `// `короткий` викинувся тут

`    `}

`    `assert\_eq!(has.lifetime, "long"); // ❌ - `короткий` все ще "borrowed" після викидання

}

Лайфтайми мають бути статично перевірені під час компіляції, а засіб перевірки запозичень Rust виконує лише дуже простий аналіз потоку керування, тому він передбачає, що кожен блок в if-else операторі і кожну підходящу відповідність в match операторі можна взяти, а потім вибирає найкоротший можливий термін життя для змінної. Як тільки змінна обмежена часом життя, вона  обмежена цим лайфтаймом *назавжди*. Тривалість життя змінної може лише скорочуватися, і все стиснення визначається під час компіляції.

Ключові тези:

- лайфтайми статично перевіряються під час компіляції
- тривалість життя не може зростати, зменшуватися чи змінюватися будь-яким чином під час виконання
- Перевірка запозичень Rust завжди вибирає найкоротший термін життя для змінної, припускаючи, що всі шляхи коду можуть бути використані

**9) переведення mut refs у  shared  є безпечним**

Помилкові твердження

- повторне запозичення посилання закінчує термін її життя та починає новий

Ви можете передати mut ref до функції, яка очікує shared ref, тому що Rust неявно повторно запозичить mut ref як незмінну:

fn takes\_shared\_ref(n: &i32) {}

fn main() {

`    `let mut a = 10;

`    `takes\_shared\_ref(&mut a); // ✅

`    `takes\_shared\_ref(&\*(&mut a)); // рядок вище,але більш уточнений (after desugaring)

}

Інтуїтивно це має сенс, оскільки немає ніякої шкоди в тому, щоб повторно запозичити mut ref як незмінне, чи не так? Як не дивно, ні, оскільки наведена нижче програма не компілюється:

fn main() {

`    `let mut a = 10;

`    `let b: &i32 = &\*(&mut a); // повторно запозичені як незмінні

`    `let c: &i32 = &a;

`    `dbg!(b, c); // ❌

}

Видає таку помилку:

error[E0502]: cannot borrow `a` as immutable because it is also borrowed as mutable

` `--> src/main.rs:4:19

`  `|

3 |     let b: &i32 = &\*(&mut a);

`  `|                     -------- mutable borrow occurs here

4 |     let c: &i32 = &a;

`  `|                   ^^ immutable borrow occurs here

5 |     dbg!(b, c);

`  `|          - mutable borrow later used here



Змінне запозичення має місце, але воно негайно і беззастережно знову запозичується як незмінне, а потім відкидається. Чому Rust ставиться до незмінного повторного запозичення так, ніби воно все ще має ексклюзивний лайфтайм mut ref? Хоча в окремому прикладі вище немає жодних проблем, дозволяючи можливість пониження версії mut refs до shared посилань(ref) дійсно створює потенційні проблеми безпеки пам’яті:

use std::sync::Mutex;

struct Struct {

`    `mutex: Mutex<String>

}

impl Struct {

`    `// понижує mut self до shared str

`    `fn get\_string(&mut self) -> &str {

`        `self.mutex.get\_mut().unwrap()

`    `}

`    `fn mutate\_string(&self) {

`        `// якщо Rust дозволяє понижувати mut refs до shared refs

`        `// тоді  рядок скасує будь-який  shared

`        `// refs повернулись із get\_string методу

`        `\*self.mutex.lock().unwrap() = "surprise!".to\_owned();

`    `}

}

fn main() {

`    `let mut s = Struct {

`        `mutex: Mutex::new("string".to\_owned())

`    `};

`    `let str\_ref = s.get\_string(); // mut ref понижений до shared ref

`    `s.mutate\_string(); // str\_ref став недійсним,тепер відокремлений вказівник

`    `dbg!(str\_ref); // ❌ - як і очікувалось!

}

Справа в тому, що коли ви повторно запозичуєте mut ref як shared ref, ви не отримуєте цього shared ref без серйозних проблем: це продовжує термін життя mut ref на час повторного запозичення, навіть якщо сам mut ref скидається. Використання повторно запозиченого shared ref дуже складне, оскільки воно незмінне, але воно не може перетинатися з іншими спільними посиланнями. Повторно запозичений shared ref має всі мінуси mut ref та всі мінуси shared ref і не має жодних плюсів. Я вважаю, що повторне запозичення mut ref як shared ref слід вважати антишаблоном Rust. Важливо знати про цей антишаблон, щоб ви могли легко помітити його, побачивши такий код:

// понижує mut T до shared T

fn some\_function<T>(some\_arg: &mut T) -> &T;

struct Struct;

impl Struct {

`    `// понижує mut self до shared self

`    `fn some\_method(&mut self) -> &Self;

`    `// понижує mut self до shared T

`    `fn other\_method(&mut self) -> &T;

}

Навіть якщо ви уникаєте повторного запозичення в сигнатурах функцій і методів, Rust все одно виконує автоматичні неявні повторні запозичення, тому легко зіткнутися з цією проблемою, не усвідомлюючи цього:

use std::collections::HashMap;

type PlayerID = i32;

#[derive(Debug, Default)]

struct Player {

`    `score: i32,

}

fn start\_game(player\_a: PlayerID, player\_b: PlayerID, server: &mut HashMap<PlayerID, Player>) {

`    `// отримати гравців із сервера або створити & вставити нових    //гравців,якщо вони ще не існують

`    `let player\_a: &Player = server.entry(player\_a).or\_default();

`    `let player\_b: &Player = server.entry(player\_b).or\_default();

`    `// робимо щось з гравцями

`    `dbg!(player\_a, player\_b); // ❌

}

Наведене вище не вдалося скомпілювати. or\_default()повертає  &mut Player який ми неявно повторно запозичуємо як &Player через наші явні анотації типу. Щоб робити те, що ми хочемо, ми повинні:

use std::collections::HashMap;

type PlayerID = i32;

#[derive(Debug, Default)]

struct Player {

`    `score: i32,

}

fn start\_game(player\_a: PlayerID, player\_b: PlayerID, server: &mut HashMap<PlayerID, Player>) {

`    `// відкидає повернуті mut Player refs ,так як ми і так не можемо //використовувати їх разом

`    `server.entry(player\_a).or\_default();

`    `server.entry(player\_b).or\_default();

`    `// знову забрати гравців, цього разу незмінно отримати їх без будь-яких //неявних повторних запозичень

`    `let player\_a = server.get(&player\_a);

`    `let player\_b = server.get(&player\_b);

`    `// щось робимо з гравцями

`    `dbg!(player\_a, player\_b); // ✅

}

Якось незручно та незграбно, але це та жертва, яку ми приносимо на вівтарі безпеки пам’яті.

Ключові тези:

- намагайтеся не позичати mut ref як shared ref, інакше вас чекають неприємності
- повторне запозичення mut ref не закінчує термін його дії, навіть якщо ref відкидається

**10) замикання дотримуються тих самих правил виключення лайфтайм-параметрів, що й функції**

Це швидше Rust розуміння ,ніж помилкове уявлення.

Замикання, незважаючи на те, що і є функціями, не відповідають тим самим правилам виключення, що і функції.

fn function(x: &i32) -> &i32 {

`    `x

}

fn main() {

`    `let closure = |x: &i32| x; // ❌

}

Видає:

error: lifetime may not live long enough

` `--> src/main.rs:6:29

`  `|

6 |     let closure = |x: &i32| x;

`  `|                       -   - ^ returning this value requires that `'1` must outlive `'2`

`  `|                       |   |

`  `|                       |   return type of closure is &'2 i32

`  `|                       let's call the lifetime of this reference `'1`



Після спрощення  коду отримуємо:

// вхідний лайфтайм застосовується до вихідного

fn function<'a>(x: &'a i32) -> &'a i32 {

`    `x

}

fn main() {

`    `// і вхідний і вихідний отримують кожен їхній чіткий лайфтайм

`    `let closure = for<'a, 'b> |x: &'a i32| -> &'b i32 { x };

`    `// замітка: рядок вище не є синтаксично правильним, але ми використовуємо його в демонстративних цілях

}

Немає вагомих причин для цієї невідповідності. Замикання спочатку були реалізовані з іншим типом припущення  семантики, ніж функції, і тепер ми застрягли на цьому назавжди, тому що їх уніфікація на цьому етапі була б критичною зміною. Отже, як ми можемо явно занотувати тип замикання? Наші варіанти включають:

fn main() {

`    `// приведення до трейт об’єкта, стає без розміру, на жаль, помилка //компіляції

`    `let identity: dyn Fn(&i32) -> &i32 = |x: &i32| x;

`    `// можна розмістити його в купі як обхідний шлях, але він виглядає //незграбним

`    `let identity: Box<dyn Fn(&i32) -> &i32> = Box::new(|x: &i32| x);

`    `// можна пропустити виділення і просто створити статичне посилання

`    `let identity: &dyn Fn(&i32) -> &i32 = &|x: &i32| x;

`    `// розгорнутий попередній рядок :)

`    `let identity: &'static (dyn for<'a> Fn(&'a i32) -> &'a i32 + 'static) = &|x: &i32| -> &i32 { x };

`    `// це було б ідеально, але це неправильний синтаксис

`    `let identity: impl Fn(&i32) -> &i32 = |x: &i32| x;

`    `// це також було б ідеально, але це також неправильний синтаксис

`    `let identity = for<'a> |x: &'a i32| -> &'a i32 { x };

`    `// так як "impl trait" працює в позиції повернення функції 

`    `fn return\_identity() -> impl Fn(&i32) -> &i32 {

`        `|x| x

`    `}

`    `let identity = return\_identity();

`    `// більш узагальнена версія попереднього рішення

`    `fn annotate<T, F>(f: F) -> F where F: Fn(&T) -> &T {

`        `f

`    `}

`    `let identity = annotate(|x: &i32| x);

}

Наскільки я впевнений, ви вже помітили з наведених вище прикладів, коли типи замикання використовуються як межі трейтів , вони дотримуються звичайних правил виключення часу життя функції.

Тут немає реального уроку чи розуміння , це просто те, що є.

Ключові тези:

- у кожній мові є проблеми🤷

**Висновки**

- T є надмножиною як  &T, так і &mut T
- &T і &mut T є множинами, що не перетинаються
- T: 'static слід читати як *" T обмежений 'static* лайфтаймом*"*
- якщо T: 'static , то T може бути запозиченим типом з 'static лайфтаймом *або* owned(власним) типом
- оскільки T: 'static включає owned типи, це означає, що T
  - можна динамічно розподіляти під час виконання
  - не має бути дійсним для всієї програми
  - може бути безпечно і вільно зміненим
  - може динамічно скидатися під час виконання
  - може мати різну тривалість життя
- T: 'a є більш загальним і більш гнучким, ніж &'a T
- T: 'a приймає власні типи, власні типи, які містять посилання
- &'a T приймає лише посилання
- якщо T: 'static тоді , T: 'a оскільки 'static >= 'a для всіх 'a
- майже весь код Rust є узагальненим, і всюди є приховані лайфтайм анотації
- Правила виключення лайфтайм-параметрів Rust для функцій не завжди підходять для кожної ситуації
- Rust не знає більше про семантику вашої програми, ніж ви
- дайте своїм життєвим анотаціям описові назви
- намагайтеся пам’ятати про те, де ви розміщуєте явні лайфтайм анотації  і чому
- всі об'єкти ознак мають певні межі тривалості життя за умовчанням
- повідомлення про помилки компілятора Rust пропонують виправлення, які змусять скомпілювати вашу програму, що не те саме, що виправлення, які змусять вас скомпілювати програму *та* найкраще відповідати вимогам вашої програми
- лайфтайми статично перевіряються під час компіляції
- тривалість життя не може зростати, зменшуватися чи змінюватися будь-яким чином під час виконання
- Перевірка запозичень Rust завжди вибирає найкоротший термін життя для змінної, припускаючи, що всі шляхи коду можуть бути використані
- намагайтеся не позичати mut ref як shared ref, інакше вас чекають неприємності
- повторне запозичення mut ref не закінчує термін його дії, навіть якщо ref відкидається
- у кожній мові є проблеми🤷



 

Корисні посилання

Оригінал статті: [Common Rust Lifetime Misconceptions](https://github.com/pretzelhammer/rust-blog/blob/master/posts/common-rust-lifetime-misconceptions.md#common-rust-lifetime-misconceptions) 

Наші соц. мережі:

- [LinkedIn](https://www.linkedin.com/company/learn-together-pro) 
- [GitHub.com](https://github.com/Learn-Together-Pro)
- [Facebook](https://www.facebook.com/learntogetherpro)
- Чат телеграм[ Вивчаємо Rust Разом](https://t.me/rustlang_ua)

Також цікавий канал англійською [Learn Rust Together](https://t.me/learn_rust), все про мову Rust(є багато цікавого для новачків і профі своєї справи).





