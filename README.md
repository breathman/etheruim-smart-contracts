## Запуск под Windows

Для запуска локальной p2p сети необходимо:

1. Установить Geth
2. Установить Mist

##### Запускаем Geth.
> geth  --dev --rpc --rpcapi \.\pipe\geth.ipc console


##### Запускаем Mist.
> C:\Program Files\Mist\Mist.exe localhost:8545


## Простейший смарт-контракт

Код смарт-контракта оборачивается в блок {}. 

```javascript
contract SimpleToken {}
```

Для проверки баланса наших токенов на адресе создаем функцию **balanceOf** как ассоциативный массив, ключи которого - адреса Etherium, а значения - количество токенов, принадлежащих данному адресу.
```javascript
mapping (address => uint256) public balanceOf;
```
Так как функция balanceOf имеет модификатор public - это значит, что проверка баланса будет доступна в интерфейсе контракта для всех желающих.

У каждого смарт-контракта есть функция конструктор, именуемая также, как и контракт. В ней задается изначальное контракта на момент его создания, например количество выпускаемых токенов (Может быть как финсированной величиной, так и задаваться при деплое контракта в блокчейн), а также другие служебные параметры (Наименование токена, символы для обозначения, количество знаков после запятой).
```javascript
string public name;
string public symbol;
uint8 public decimals;

function SimpleContract(uint256 initialSupply, string tokenName, string tokenSymbol, uint8 decimalUnits) {
    balanceOf[msg.sender] = initialSupply;
    name = tokenName;
    symbol = tokenSymbol;
    decimals = decimalUnits;
}
```
В нашем случае в интерфейсе кошелька при деплое контракта будет предложено задать значения основных параметров. Все выпущенные токены мы передаем создателю контракта.

Центральной функцией в смарт-контракте является функция перевода токенов с одного адреса на другой.
```javascript
event Transfer(address indexed from, address indexed to, uint256 value);
function transfer(address _to, uint256 _value) {
    require(balanceOf[msg.sender] >= _value && balanceOf[_to] + _value >= balanceOf[_to]);
    balanceOf[msg.sender] -= _value;
    balanceOf[_to] += _value;
    Transfer(msg.sender, _to, _value);
}
```
В параметрах указывается на какой адрес будут перечислены токены и в каком количестве.
В качестве валидатора используем конструкцию ***require***, которая проверяет логическое выражение на соответствие ***true***. В данном случае мы проверяем, что баланс кошелька, от имени которого выполняется контракт позволяет провести операцию, и проверяем кошелек адресата от снятия средств путём указания отрицательного значения количества токенов. 
После проведения операций с балансами адресов, мы инициируем событие ***Transfer***, которое может отслеживаться кошельками послего его объявления.

##### *Оптимизация*
*Существует альнернативная контрукция assert, которая проверяет выражение на false. Ее не рекомендуется использовать там, где это возможно, т.к. в случае исключения будет использован весь доступный газ*

##### *Важно*
*Нельзя отправить эфир на адрес контракта. Транзакция не пройдет, токены эфира не будут списаны с отправителя, но газ будет израсходован*

*При деплое контракта, также не стоит посылать эфир. Контракт не установится, эфир не будет списан, газ будет потрачен.*

## Централизованное администрирование

В децентрализованных приложениях по определению нет управляющего узла, но это не значит что мы не можем ввести управляющую структуру в рамках нашего смарт-контракта. 
Администратор контракта может выполнять функции выпуска новых токенов, управления доступом к токенам и др., им может быть как простой аккаунт, так и другой смарт-контракт. 
Функцию эмиссии токенов можно передать смарт-контракту для открытости и для того, чтобы ограничить возможности создателя.

```javascript
    contract owned {
        address public owner;

        function owned() {
            owner = msg.sender;
        }

        modifier onlyOwner {
            require(msg.sender == owner);
            _;
        }

        function transferOwnership(address newOwner) onlyOwner {
            owner = newOwner;
        }
    }
```
В приведенном примере мы вводим переменную **owner**, хранящую адрес владельца контракта, в определяем, что его владельцем является создатель.
Функция **transferOwnership()** передает права от текущего владельца к новому адресу, указанному в единственном аргументе.
Данная функция должна быть доступна только для текущего владельца, поэтому к ней применяется модификатор onlyOwner, который проверяет, является ли адрес, от имени которого выполняется контракт текущим владельцем.

##### *Важно*
*Символ подчеркивания "**_**" используется для смены потока выполнения программы. После проверки условия и выполнения тела функции поток управления будет возвращен в тело модификатора после оператора* "**_**".

*В модификаторе доступны все компоненты области видимости функции, но переменные, определенные в модификатореа в функции не видны, поскольку они могут переопределяться и нарушать контекст выполнения.*

У контрактов есть очень полезное свойтво - **наследование**. Наследование позволяет контракту получать свойства и методы других контрактов без переопределения.
Для того, чтобы контракт унаследовал свойства и методы другого контракта, необходимо использовать следующую конструкцию:
```javascript
    contract SimpleContract is owned {
        // ...
    }
```
Данная инструкция означает, что при выполнении **SimpleContract** будут доступны свойства и методы, помеченные модификатором доступа **public** (по умолчанию).