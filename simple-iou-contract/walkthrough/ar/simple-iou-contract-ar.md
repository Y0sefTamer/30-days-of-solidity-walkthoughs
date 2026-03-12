<div dir="rtl">

# عقد IOU بسيط: بناء تتبع الديون على السلسلة

مرحباً بك في تطبيق عملي واقعي: **تتبع الديون بين الأصدقاء**. اليوم، تبني عقداً ذكياً يحل المشكلة القديمة "من يدين لمن" — بشفافية كاملة وبدون أي نزاعات.

## مشكلة تقسيم الفاتورة

**السيناريو:**

- أنت و5 أصدقاء تخرجون لتناول العشاء
- الفاتورة 300 دولار، أنت تدفعها全部
- كل شخص يقول "سأحول لك لاحقاً"
- يمر أسبوعان...
- "انتظر، هل دفعت لك بالفعل؟"

**المشاكل:**

- الذاكرة تتلاشى
- لا يوجد سجل واضح
- إحراج لتذكير الناس
- تنشأ النزاعات
- تتوتر الصداقات

**حل Web3:** ضع الأمر على البلوكشين. غير قابل للتغيير، شفاف، لا جدال فيه.

## ما الذي نبنيه

نظام دفتر حسابات جماعي مع:

- **سجل الأصدقاء** - يمكن فقط للأعضاء الموافق عليهم المشاركة
- **تتبع الديون** - سجل واضح لمن يدين لمن
- **محفظة مشتركة** - إيداع ETH لتسوية سهلة
- **مدفوعات فورية** - دفع الديون بمعاملة واحدة
- **طرق تحويل متعددة** - تعلّم الفرق بين `transfer()` و `call()`

**استخدامات واقعية:**

- مجموعات الأصدقاء التي تقسم المصاريف
- تتبع إيجار/فواتير الزملاء في السكن
- إدارة نفقات السفر
- تنسيق هدايا المجموعة
- IOUs للأعمال الصغيرة

## متغيرات الحالة: ماذا نحتفظ به؟

قبل أن يعمل أي منطق، يحتاج العقد إلى تذكر بعض الأمور.

### owner

```
address public owner;
```

المالك هو من نشر العقد. هذا الشخص سيكون له أذونات خاصة، مثل إضافة أصدقاء جدد إلى المجموعة. إنه يشبه أول شخص ينشئ مجموعة Splitwise — هو من يدير الوصول.

### registeredFriends و friendList

```
mapping(address => bool) public registeredFriends;
address[] public friendList;
```

نريد أن يكون هذا العقد خاصاً بمجموعتكم. ليس أي شخص على الإنترنت يجب أن يتمكن من الانضمام والبدء في تسجيل IOUs.

إليك كيف ندير ذلك:

- `registeredFriends`: mapping للتحقق مما إذا كان العنوان جزءاً من المجموعة.
- `friendList`: array لتتبع عناوين الجميع حتى نتمكن من عرضها.

عندما يتم إضافة صديق، يذهب عنوانه إلى كليهما.

### balances

```
mapping(address => uint256) public balances;
```

الآن دعنا نتعامل مع المال. يمكن لكل شخص إيداع ETH في رصيده الشخصي داخل العقد. يمكن استخدام هذا ETH لاحقاً لسداد الديون أو إرساله إلى أصدقاء آخرين.

### debts

```
mapping(address => mapping(address => uint256)) public debts;
```

هنا يحدث السحر الحقيقي لـ IOU. هذا `mapping` متداخل، يعني:

```
debts[debtor][creditor] = amount;
```

على سبيل المثال:

```
debts[0xAsha][0xRavi] = 1.5 ether;
```

يعني أن Asha تدين لـ Ravi بمقدار 1.5 ETH. وهو موجود هناك، مخزن بشفافية على السلسلة. لا سوء فهم، لا نسيان.

## الـ Constructor

```
constructor() {
    owner = msg.sender;
    registeredFriends[msg.sender] = true;
    friendList.push(msg.sender);
}
```

عند نشر العقد: يصبح الشخص الذي نشره (`msg.sender`) هو المالك، ويُسجَّل تلقائياً كصديق، ويُضاف إلى القائمة.

## Modifiers: التحكم في الوصول

### onlyOwner

```
modifier onlyOwner() {
    require(msg.sender == owner, "Only owner can perform this action");
    _;
}
```

يضمن أن المالك فقط يمكنه تنفيذ إجراءات إدارية مثل إضافة صديق.

### onlyRegistered

```
modifier onlyRegistered() {
    require(registeredFriends[msg.sender], "You are not registered");
    _;
}
```

فقط الأصدقاء المسجلون يمكنهم استخدام دوال العقد.

## الدوال: ماذا يمكن للأصدقاء أن يفعلوا؟

### addFriend

```
function addFriend(address _friend) public onlyOwner {
    require(_friend != address(0), "Invalid address");
    require(!registeredFriends[_friend], "Friend already registered");

    registeredFriends[_friend] = true;
    friendList.push(_friend);
}
```

### depositIntoWallet

```
function depositIntoWallet() public payable onlyRegistered {
    require(msg.value > 0, "Must send ETH");
    balances[msg.sender] += msg.value;
}
```

كيف يضيف المستخدم ETH إلى رصيده داخل التطبيق.

### recordDebt

```
function recordDebt(address _debtor, uint256 _amount) public onlyRegistered {
    require(_debtor != address(0), "Invalid address");
    require(registeredFriends[_debtor], "Address not registered");
    require(_amount > 0, "Amount must be greater than 0");

    debts[_debtor][msg.sender] += _amount;
}
```

إذا كان صديقك يدين لك مالاً، تنادي هذه الدالة لتسجيل الدين.

### payFromWallet

```
function payFromWallet(address _creditor, uint256 _amount) public onlyRegistered {
    require(_creditor != address(0), "Invalid address");
    require(registeredFriends[_creditor], "Creditor not registered");
    require(_amount > 0, "Amount must be greater than 0");
    require(debts[msg.sender][_creditor] >= _amount, "Debt amount incorrect");
    require(balances[msg.sender] >= _amount, "Insufficient balance");

    balances[msg.sender] -= _amount;
    balances[_creditor] += _amount;
    debts[msg.sender][_creditor] -= _amount;
}
```

سدّد دينك باستخدام رصيد ETH المخزن داخل العقد.

### transferEther — إرسال ETH باستخدام `transfer()`

```
function transferEther(address payable _to, uint256 _amount) public onlyRegistered {
    require(_to != address(0), "Invalid address");
    require(registeredFriends[_to], "Recipient not registered");
    require(balances[msg.sender] >= _amount, "Insufficient balance");

    balances[msg.sender] -= _amount;
    _to.transfer(_amount);
    balances[_to] += _amount;
}
```

### transferEtherViaCall — إرسال ETH باستخدام `call()`

```
function transferEtherViaCall(address payable _to, uint256 _amount) public onlyRegistered {
    require(_to != address(0), "Invalid address");
    require(registeredFriends[_to], "Recipient not registered");
    require(balances[msg.sender] >= _amount, "Insufficient balance");

    balances[msg.sender] -= _amount;

    (bool success, ) = _to.call{value: _amount}("");
    balances[_to] += _amount;
    require(success, "Transfer failed");
}
```

### withdraw

```
function withdraw(uint256 _amount) public onlyRegistered {
    require(balances[msg.sender] >= _amount, "Insufficient balance");

    balances[msg.sender] -= _amount;

    (bool success, ) = payable(msg.sender).call{value: _amount}("");
    require(success, "Withdrawal failed");
}
```

اسحب ETH الخاص بك من العقد.

### checkBalance

```
function checkBalance() public view onlyRegistered returns (uint256) {
    return balances[msg.sender];
}
```

---

## فهم المappings المتداخل (Nested Mappings)

```solidity
mapping(address => mapping(address => uint256)) public debts;
```

**هذا mapping داخل mapping!**

**فكّر فيه كأنه جدول بيانات:**

```
           Ravi    Asha    Bob
Ravi       0       50      0
Asha       0       0       100
Bob        30      0       0
```

**القراءة:** `debts[Bob][Ravi] = 30` يعني "Bob يدين لـ Ravi بمقدار 30 ETH"

**لماذا المتداخل؟** نحتاج لتتبع الديون بين كل زوج من الأشخاص.

---

## transfer مقابل call: فهم تحويلات ETH

### الطريقة 1: `transfer()`

```solidity
_to.transfer(_amount);
```

**الخصائص:**

- مخصص غاز ثابت 2300
- يرجع (revert) عند الفشل
- بسيط وآمن
- **المشكلة:** قد يفشل إذا كان المستلم عقداً يتطلب استدعاءً مكلفاً

### الطريقة 2: `call()`

```solidity
(bool success, ) = _to.call{value: _amount}("");
require(success, "Transfer failed");
```

**الخصائص:**

- يمرّر كل الغاز المتاح
- يُرجع قيمة منطقية تشير للنجاح
- أكثر مرونة
- **موصى به:** الممارسة الحديثة الأفضل

**لماذا `call()` أفضل:**

- يعمل مع العقود التي تحتاج غازاً أكثر
- يمنحك تحكماً في التعامل مع الفشل
- مستقبلي (EIP-1884 كسر `transfer()`)

---

## المفاهيم الرئيسية التي تعلمتها

**1. المappings المتداخلة** - تتبع العلاقات بين كيانات متعددة

**2. التحكم في الوصول** - `onlyOwner` و `onlyRegistered` modifiers

**3. التعامل مع ETH** - الإيداع، السحب، التحويل

**4. تتبع الديون** - تسجيل وتسوية IOUs

**5. طرق التحويل** - أنماط `transfer()` مقابل `call()`

**6. إدارة المجموعات** - السجل والأذونات

---

## اعتبارات أمانية

### 1. الحماية من إعادة الدخول (Reentrancy)

**الكود الحالي قابل للهجوم:**

```solidity
function withdraw(uint256 _amount) public onlyRegistered {
    require(balances[msg.sender] >= _amount);
    balances[msg.sender] -= _amount;
    (bool success, ) = payable(msg.sender).call{value: _amount}("");
    require(success);
}
```

**لماذا آمن؟** تم تحديث الحالة قبل الاستدعاء الخارجي (نمط Checks-Effects-Interactions).

### 2. نقص عدد صحيح (Integer Underflow)

**Solidity 0.8+** لديها حماية تلقائية من overflow/underflow، لكن كن حذراً:

```solidity
balances[msg.sender] -= _amount; // Will revert if insufficient
```

### 3. التحكم في الوصول

**فقط الأصدقاء المسجلون يمكنهم:**

- تسجيل الديون
- إجراء المدفوعات
- سحب الأموال

**فقط المالك يمكنه:**

- إضافة أصدقاء جدد

---

## التطبيقات الواقعية

### Splitwise على السلسلة

**Splitwise التقليدية:**

- قاعدة بيانات مركزية
- تثق في الشركة
- يمكن إيقافها

**الإصدار على البلوكشين:**

- لامركزي
- لا يحتاج ثقة
- سجل دائم

### إدارة خزينة DAO

**حالة الاستخدام:** تتبع القروض الداخلية بين أعضاء DAO

```solidity
// DAO member borrows from treasury
recordDebt(daoMember, 1000 ether);

// Member pays back
payFromWallet(daoTreasury, 1000 ether);
```

### فواتير الأعمال

**حالة الاستخدام:** عمل صغير يتتبع IOUs للعملاء

```solidity
// Customer owes for service
recordDebt(customer, serviceAmount);

// Customer pays
payFromWallet(business, serviceAmount);
```

---

## تحدَّ نفسك

وسع هذا النظام:

1. أضف فائدة على الديون المتأخرة
2. نفّذ وظيفة مسامحة الديون
3. أنشئ آلية حل نزاعات
4. أضف دعم عملات متعددة (توكنات ERC20 مختلفة)
5. ابنِ نظام سمعة يعتمد على تاريخ السداد

لقد أتقنت تتبع الديون وإدارة الشؤون المالية الجماعية على السلسلة!

</div>
