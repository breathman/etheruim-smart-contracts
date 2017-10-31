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

