# Zero Node

["Zero Node"](images/preview.jpg)

Zero Node is a standalone Meshtastic repeater based on the RP2040 Zero microcontroller and the Ra-01/Ra-02 radio modules. It has a CN3791 module for charging from the solar pantels, as well as a TP4056 for charging from an external power supply if one is provided. The low consumption of the repeater (~20 mA in idle mode) will ensure operation for 100-250 hours with a 6000 mAh battery without any recharging. With a small solar panel, the operating time increases radically - a 3 W solar panel can almost completely compensate for the repeater's consumption even on winter sunless days. Paired with a good, high-mounted antenna, this repeater will be an excellent solution for expanding the coverage of your Meshtastic network!

["PCB"](images/pcb.jpg)
["Case"](images/case.jpg)

## Microcontroller firmware

In order to flash the RP2040 microcontroller, it is enough to simply hold down the BOOT button on the microcontroller, and then connect it to the computer. The microcontroller will connect as a flash drive, to which you need to drop a special **.uf2** file. **.uf2** files with the firmware for this repeater are provided in the "firmware" directory of this git repository. There are two firmwares types - *fast-cpu* and *slow-cpu*. When each of them is needed is described in the next section.

## Microcontroller power consumption

By default, the microcontroller consumes about 40 mA of current on the 3.3V line. We can reduce this consumption by half by reducing the microcontroller clock frequency - from 133 MHz to 18 MHz. However, in this case, two problems arise - first, USB stops working (however, the microcontroller can still be flashed via USB), and therefore it is necessary to use an external UART converter to interact with the repeater (pins 0 - Tx, 1 - Rx). Secondly, personally, in my case, the repeater stops executing commands via UART - it receives them, responds, but does not execute them, so you cannot configure the repeater in this way. To get around this problem, you can first configure the node on the firmware with a normal clock frequency, and only after that upload the firmware with a reduced clock frequency - the settings are stored in the microcontroller's memory, so everything will work as it should.

Therefore, there are two firmware in the releases:

- fast-cpu - firmware with a normal clock frequency, for configuring the repeater.
- slow-cpu - firmware with a reduced clock frequency, for using the repeater.

It should also be noted that at 3.4V on the battery, the repeater will go into a deep sleep - Super Deep Sleep, so be sure to configure its duration. RP2040 does not know how to exit SDS, so it will simply reboot when time comes.

## Case for the repeater

The repeater board should fit most moisture-proof cases measuring 115x90 mm. Personally, I used the case **[11-3 from Sanhe](https://www.rcscomponents.kiev.ua/product/11-3_121626.html)**. Judging by the drawings, the G311MF and G221CMF from **Gainta** should also fit. Below are links to the cases and their drawings:

- [11-3 Sanhe](https://www.rcscomponents.kiev.ua/product/11-3_121626.html) [(drawing)](https://www.rcscomponents.kiev.ua/datasheets/11-3-datasheet.pdf)
- [G311MF](https://www.rcscomponents.kiev.ua/product/g311mf_105942.html) [(drawing)](https://www.rcscomponents.kiev.ua/datasheets/g212mf_g311mf.pdf)
- [G221CMF](https://www.rcscomponents.kiev.ua/product/g221cmf-gainta-korpus-pc-z-prozoroiu-kryshkoiu-svitlo-siryi-115kh90kh80mm_55570.html) [(drawing)](https://www.rcscomponents.kiev.ua/datasheets/g221mf_g331mf.pdf)

## Compiling the firmware

To compile the firmware for this repeater, I created a separate variant file and also slightly modified the Meshtastic code. Since the changes are not very significant, I did not create a separate fork of the Meshtastic repository, but instead made a git patch. So to compile the firmware you need to clone the Meshtastic repo, open it, download my patch, and apply it:

```bash
git clone https://github.com/meshtastic/firmware
cd firmware
wget
git apply -v zero-node.patch
```

After that, the firmware can be compiled in a way convenient for you - via VS Code + PlatformIO, or via [PlatformIO Core](https://so1der.github.io/articles/pio.html#pio). The patch added a new PlatformIO environment called zero-node, so before compiling you need to select it, either in the general platformio.ini file:

```ini
[platformio]
default_envs = zero-node
```

Or at the compilation stage using PlatformIO Core:

```bash
pio run -e zero-node
```

After compiling, the **.uf2** file will be available at the path `firmware/.pio/build/zero-node/`. You can manually "drag" it to the microcontroller, or you can directly in VS Code click the "Upload" checkbox (Ctrl + Alt + U), or run the command:

```bash
pio run -e zero-node --target upload
```

The main board settings are made in a special `variant.h` file of the board at the path: `variants/rp2040/zero-node/variant.h`. By default, you compile the firmware with a normal clock speed. To compile the firmware with a reduced clock speed, uncomment the corresponding define at the beginning of the file.

Normal clock speed:
```h
// Reduce clock speed down to 18 MHz to reduce power consumption
//#define RP2040_SLOW_CLOCK
```

Reduced clock speed:
```h
// Reduce clock speed down to 18 MHz to reduce power consumption
#define RP2040_SLOW_CLOCK
```


## Schematic 

["Schematic"](images/schematic.png)

Key points regarding the schematic:

- You must use **TLV1117LV33** as a voltage stabilizer. AMS1117 will not work, as it has a much larger voltage drop, and will not be able to effectively use the battery.
- Diode D1 should be replaced with a jumper, because it prevents the MPPT module from working normally, as it makes feedback with the battery impossible.
- LED D3 is needed only for debugging, in the slow-cpu firmware it hangs on the Tx line of the microcontroller, and therefore will be constantly lit, so you can not solder it and save a couple of milliamperes.
- U.FL connector J1 is needed only if you use the Ra-01 radio module


# Zero Node

["Zero Node"](images/preview.jpg)

Zero Node - це автономний Meshtastic-ретранслятор, побудований на основі мікроконтролера RP2040 Zero, та радіомодулів Ra-01/Ra-02. Він має модуль CN3791 для зарядки від сонця, а також TP4056 для зарядки від зовнішнього блока живлення якщо такий передбачено. Низьке споживання ретранслятора (~20 мА в стані спокою) забезпечить роботу впродовж 100-250 годин з акумулятором ємністю 6000 мАг без будь якої підзарядки. За наявності невеличкої сонячної панелі час роботи радикально підвищується - сонячна панель на 3 Вт здатна майже повністю компенсувати споживання ретранслятора навіть в зимні безсонячні дні. В парі з гарною, високо розташованою антеною, даний ретранслятор стане чудовим рішенням для розширення покриття вашої Meshtastic мережі!

["PCB"](images/pcb.jpg)
["Case"](images/case.jpg)

## Прошивка мікроконтролера

Для того щоб прошити мікроконтролер RP2040, достатньо всього лише затиснути кнопку BOOT на мікроконтролері, після чого підключити його до комп'ютера. Мікроконтролер підключиться у вигляді флеш накопичувача, на який треба скинути спеціальний **.uf2** файл. **.uf2** файли з прошивкою для даного ретранслятора знаходяться в директорії "firmware" даного гіт репозиторія. Там міститься два типи прошивки - *fast-cpu* та *slow-cpu*. Навіщо потрібна кожна з них - описано в наступному розділі.

## Споживання мікроконтролера

За замовчуванням мікроконтролер споживає близько 40 мА струму по 3.3V лінії. Ми маємо змогу зменшити це споживання вдвічі, зменшивши тактову частоту мікроконтролера - з 133 МГц до 18 МГц. Однак в такому випадку випливає дві проблеми - по перше, перестає працювати USB (однак мікроконтролер все ще можна прошити по USB), а отже треба використовувати зовнішній UART перетворювач для взаємодії з ретранслятором (піни 0 - Tx, 1 - Rx). По друге - особисто в мене ретранслятор в такому випадку перестає виконувати команди по UART - він їх приймає, відповідає, але не виконує, тому ретранслятор таким чином не налаштуєш. Щоб обійти дану проблему, можна спочатку налаштувати вузол на прошивці з звичайною тактовою частотою, і вже після цього залити прошивку з зниженою тактовою частотою - налаштування зберігаються в пам'яті мікроконтролера, тому все працюватиме як слід.

Тому в релізах присутні дві прошивки:

- fast-cpu - прошивка зі звичайною тактовою частотою, для налаштування ретранслятора.
- slow-cpu - прошивка зі зменшеною тактовою частотою, для використання ретранслятора.

Також слід зауважити, що при 3.4V на акумуляторі, ретранслятор піде в глибокий сон - Super Deep Sleep, тому обов'язково налаштуйте його тривалість. RP2040 не вміє виходити з SDS, тому він просто перезавантажиться.

## Корпус для ретранслятора 

Плата ретранслятора повинна підійти для більшості вологозахищених корпусів розміром 115х90 мм. Особисто я використав корпус **[11-3 від Sanhe](https://www.rcscomponents.kiev.ua/product/11-3_121626.html)**. Якщо судити по кресленням, то повинні підійти ще G311MF та G221CMF від **Gainta**. Нижче наведені посилання на корпуси та їх креслення:

- [11-3 Sanhe](https://www.rcscomponents.kiev.ua/product/11-3_121626.html) [(креслення)](https://www.rcscomponents.kiev.ua/datasheets/11-3-datasheet.pdf)
- [G311MF](https://www.rcscomponents.kiev.ua/product/g311mf_105942.html) [(креслення)](https://www.rcscomponents.kiev.ua/datasheets/g212mf_g311mf.pdf)
- [G221CMF](https://www.rcscomponents.kiev.ua/product/g221cmf-gainta-korpus-pc-z-prozoroiu-kryshkoiu-svitlo-siryi-115kh90kh80mm_55570.html) [(креслення)]()https://www.rcscomponents.kiev.ua/datasheets/g221mf_g331mf.pdf

## Компіляція прошивки

Для компіляції прошивки для даного ретранслятора я створював окремий variant файл, а також трохи модифікував код Meshtastic. Через те, що зміни не дуже значні, я не створював окремий форк репозиторія Meshtastic, а натомість зробив git патч. Тож для компіляції прошивки треба клонувати репо Meshtastic, відкрити його, завантажити мій патч, та застосувати його:

```bash
git clone https://github.com/meshtastic/firmware
cd firmware
wget 
git apply -v zero-node.patch
```

Після цього прошивку можна буде компілювати зручним для вас способом - через VS Code + PlatformIO, або виключно через [PlatformIO Core](https://so1der.github.io/articles/pio.html#pio). Патч додав новий PlatformIO environment під назвою zero-node, відповідно перед компіляцією треба обрати його, або в загальному platformio.ini файлі:

```ini
[platformio]
default_envs = zero-node
``` 

Або на етапі компіляції за допомогою PlatformIO Core:

```bash
pio run -e zero-node
```

Після компіляції **.uf2** файл буде доступний за шляхом `firmware/.pio/build/zero-node/`. Можна вручну "перетягнути" його на мікроконтролер, або можна прямо в VS Code натиснути галочку "Upload" (Ctrl + Alt + U), чи виконати команду:

```bash
pio run -e zero-node --target upload
```

Основні налаштування плати відбуваються в спеціальному `variant.h` файлі плати за шляхом: `variants/rp2040/zero-node/variant.h`. За замовчуванням ви скомпілюєте прошивку зі звичайною тактовою частотою. Щоб скомпілювати прошивку зі зменшеною тактовою частотою, розкоментуйте відповідний дефайн на початку файла.

Звичайна тактова частота:
```h
// Reduce clock speed down to 18 MHz to reduce power consumption
//#define RP2040_SLOW_CLOCK
```

Зменшена тактова частота:
```h
// Reduce clock speed down to 18 MHz to reduce power consumption
#define RP2040_SLOW_CLOCK
```


## Схема

["Schematic"](images/schematic.png)

Основні моменти стосовно схеми:

- В якості стабілізатора треба обов'язково використовувати **TLV1117LV33**. AMS1117 не підійде, так як має значно більше падіння напруги, і не зможе ефективно використовувати акумулятор.
- Діод D1 слід замінити на перемичку, бо він заважає MPPT модулю нормально працювати, так як унеможливлює зворотній зв'язок з акумулятором.
- Світлодіод D3 потрібен лише для відладки, в прошивці slow-cpu він висить на Tx лінії мікроконтролера, а отже буде постійно світитись, тож можна його не запаювати і зекономити пару міліампер.
- U.FL конектор J1 потрібен лише якщо ви використовуєте радіомодуль Ra-01


