# Програмна частина для «Тривимірного пристрою з механічною розгорткою - 3D Глобус»

### Вимоги до ПЗ:
- розпізнавати контакт на фотодіоді, для визначення положення ряду світлодіодів у просторі та розрахунку часу одного оберту;
- розрахунок часу одного оберту ряду світлодіодів, для визначення частоти зміни «кадрів» зображення;
- записати бітмап зображення у флеш пам’ять мікроконтроллера;
- виводити на світлодіоди бітмап зображення записане у флеш пам’ять;
- атвоматично підлаштовувати швидкість виведення зображення під швидкість обертання;
- забезпечити можливість зміни напряму «обертання» зображення;
Програма повинна працювати на мікроконтролері Attiny2313 з підключеним кварцовим резонатором, що працює з частотою 20 МГц.

### Для реалізації використано:
- C 
- <avr/io.h>
- IDE Atmel Studio
- CAD Proteus Design Suite
### Опис алгоритму

<p> <pre>За допомогою директиви препроцесора #define задаємо константи:
    F_CPU - вказує компілятору значення частоти мікропроцесора, або кварцевого резонатора
    column - зберігає кількість стовпців, на які поділене зображення.
    image - бітмап зображення, 
      яке за допомогою ключового слова __flash ініціалізується у Flash пам’яті мікроконтролера. <br>
  Далі ініціалізуються змінні:
    - count -  іттератор  п’яти байт кожного стовпця зображення;
    - start - зберігає індекс байта з якого почнеться наступний стовпець;
    - temp - зберігає один байт зображення за певним індексом;
    - time - в цю змінну розраховується час відображення кожного стовпця;
    - reset - підраховує кількість виконаних оборотів;
    - i -  іттератор.<br>
   
 Налаштовуєм порти:
    - DDRB = 0b00000111; - перші 3 виходи порту B налаштовуються на вивід даних;
    - DDRD = 0x00; - порт D повністю налаштовується на введення даних, 
      так як лише 4 вивід порта D має підключений фотодіод.<br>
        
        
Весь алгоритм програми повинен виконуватись в нескінченному циклі, 
щоб мікроконтролер працював постійно і не зупинявся, поки ми не відключимо живлення.
Змінна reset за замовчуванням має значення 250, тому відразу ж збільшуємо її на 1, щоб виконалась умова if(reset>250).</pre>
<pre>Якщо умова виконується:<br>
    - обнуляємо reset;
    - викликаємо функцію set_time(), в якій:<br>
         - TCCR1B=0x00; - явно вказуємо що таймер вимкнено;
         - в рахівний регістр TCNT1 записуємо 0;
         - чекаємо доки на фотодіод буде впливати світло, дізнаємося про це за допомогою функції button():<br>
                - подаємо на вихід, до якого підключений фотодіод, високий логічний рівень;
                - записуємо значення з регістра PIND у змінну;
                - var>>=2; - виконуємо зрошування вмісту змінної на два біти вправо;
                - перемножуємо вміст змінної var та 0b0000001;
                - повертаємо значення змінної var;<br>
          - запускаємо лічильник зі встановленим режимом  переддільника на 64;
          - знову чекаємо доки на фотодіод буде впливати світло;
          - зупиняємо таймер;
          - повертаємо значення регістра TCNT1;<br>
      - результат виконання функції set_time() записуємо у змінну time;
      - розділяємо вміст змінної time на кількість стовпців зображення;
      - по бажанню множимо time на 0.98, для ефекту обертання зображення, за часовою стрілкою;
      - у змінну start записуємо 0;
      - починається цикл, в якому по черзі виводяться виводяться всі стовпці зображення, а точніше:<br>
            - запускається лічильник з переддільником на 64;
            - починається цикл, який виконується п’ять разів, так як у нас п’ять зсувних регістрів;
            - починається цикл, для побітового виведення стовпця;
            - temp=image[count+start]; - у змінну записується один байт зображення за індексом, 
              який визначається сумою іттератора п’яти байт кожного стовпця та номером байта наступного стопця; 
            - змінна temp домножується на зрошене вправо число 0b10000000, 
              кількість біт на яку зрошується число дорівнює значенню іттератора циклу; 
            - якщо значення змінної temp дорівнює числу 0b10000000 зрошеному вправо, 
              кількість біт на яку зрошується число дорівнює значенню іттератора циклу, 
              на 3 вивід порту  подається високий логічний рівень, інакше - низький; 
            - закінчилися два останні цикли;
            - змінна start збільшується на 5, але якщо start==column*5, у змінну записується нуль;
            - подаємо короткочасний імпульс на другу ніжку порту B, до якої підключений вхід ST_CP зсувного регістра. 
            - очікуємо доки відрахується час, розрахований на вивід кожного стовпця;<br>
      - кінець циклу, в якому виводяться всі стовпці зображення;
      - після чого все повторюється.  
</pre>
</p><br><br>

### Опис математичного методу рішення задачі

<pre>
Програмування мікроконтролерів практично цілком зав’язане на операціях с бітами, таких як:
  <<, >> - бітовий зсув, здвигає біти у змінній вправо, або вліво на вказану кількість;
  & - бітове множення (И);
  | - бітове АБО;
  ^ - бітове виключне АБО;
  ~ - бітова інверсія.
  </pre>
  
  ## Основний алгоритм виведення зображення працює наступним чином:
  <pre>
    - У змінну temp запишемо певний байт з масиву image[], для прикладу нехай буде temp = 0b00011000;
    
    - Щоб дізнатись значення кожного біту це число ми будемо вісім разів домножувати на (0b100000000>>i), 
    де i – іттератор, що збільшується від 0 до 8. 
          Наприклад, на шостій іттерації циклу і буде дорівнювати п’яти, 
          тому число 0b100000000 зсунеться на 5 біт вправо і в результаті буде  виглядати так 0b00001000. 
          Після чого число у змінній temp, в саме 0b00011000, домножується  на 0b00001000, 
          в результаті чого отримаємо число 0b00001000
          
          
      - Тепер визначимо, яке число знаходиться у п’ятому біті змінної temp. 
      
        Порівняємо вміст змінної temp з (0b100000000>>i), де i – іттератор, що збільшується від 0 до 8. 
        Так як ми вважаємо, що проходить  шоста іттерація циклу  - змінна і дорівнює 5, тобто маємо число 0b00001000. 
        Таким чином маємо порівняння: if(0b00001000==0b00001000). 
        В даному випадку порівняння вірне, тому оператор if повертає true, та виконує наступну операцію: PORTB|=(1<<2)
        
      - Операція PORTB|=(1<<2) визначає, що на третій вивід порту B необхідно подати високий логічний рівень (1)
        Це досягається за допомогою операції АБО, за допомогою якої можна змінювати певні біти не змінюючи інші
        Результатом операції є число 0b00000100, тобто на третій вивід подаємо одиницю.
        Проте може статися так що порівняння виявиться невірним і оператор if поверне false. 
        Тоді виповниться операція PORTB&=~(1<<2), що буде означати нуль на третьому виводі; 

      - Ми вже знаємо що операція 1<<2 дорівнює числу 0b00000100, 
        однак перед цією операцією стоїть знак інверсії тому число буде інвертовано та матиме наступний вигляд: 0b111110111. 
        Знову ж, припустимо що на всіх виходах порту B ми маємо нулі, тоді в результаті операції множення отримали число 0b00000000, 
        це означає, що на третій вивід порту B подається низький логічний рівень, тобто нуль.

  
  </pre>
  
  



<img src=" Project_files/Image/1.png" width="300px">  <img src=" Project_files/Image/prilad.jpeg" width="300px">  <img src=" Project_files/Image/2.jpg" width="300px"> 
<img src=" Project_files/Image/Proteus.png" width="600px"> 



