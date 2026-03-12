<div dir="rtl">

# البنك الصغير لـ Ether: التعامل مع ETH الحقيقي

مرحباً بك في مرحلة مهمة: **التعامل مع Ether الحقيقي**. اليوم ستتعلم الفرق بين تتبع الأرقام وإدارة العملة الحقيقية — الأساس لكل بروتوكول DeFi.

## من الأرقام إلى المال

**حتى الآن، لقد بنيت:**

- أنظمة مزاد (تتبع العروض)
- صناديق كنز (التحكم في الوصول)
- متتبعات رصيد (mappings)

**لكن الحقيقة:** كل ذلك كان مجرد أرقام مخزّنة. لم يتحرك مال حقيقي.

**اليوم يغير كل شيء.** أنت الآن تبني عقداً يقوم بـ:

- استقبال ETH الحقيقي
- تخزينه بأمان
- تتبع الملكية
- إرساله مرة أخرى عند الطلب

**هذا هو أساس:**

- بروتوكولات DeFi (أكثر من 100 مليار دولار مقفلة)
- أنظمة الدفع
- حسابات التوفير
- صناديق الاستثمار

تخيل أنك وأصدقاؤك المقربون تريدون إنشاء تجمع توفير مشترك — بنك توفير رقمي. ليس مفتوحاً للجميع، بل نادي خاص بكم.

يجب أن يكون بإمكان الجميع:

- الانضمام إلى المجموعة (إذا تمت الموافقة)
- إيداع المدخرات
- التحقق من رصيدهم
- وربما حتى السحب عند الحاجة

والأجمل؟ في النهاية، سيصبح هذا البنك الصغير قادراً على قبول ETH حقيقي. صحيح — Ether فعلي، مخزن بأمان على السلسلة.

لنبدأ البناء.

## ما البيانات التي يحتاجها بنكنا الصغير؟

قبل أن نبدأ بكتابة الدوال، دعنا نفكر في الأمر كما لو أننا نصمم نظاماً من الصفر. نجلس حول طاولة ونسأل:

"ما الذي يجب أن يتذكره هذا البنك الصغير؟"

### 1. من هو المسؤول؟

كل مجموعة تحتاج قائداً — شخصاً مسؤولاً عن إضافة الأعضاء الجدد، والحفاظ على النظام.

لذلك نعرّف مدير البنك:

```
address public bankManager;
```

هذا هو الشخص الذي نشر العقد. لديه صلاحيات المشرف — الوحيد الذي يستطيع الموافقة على أعضاء جدد للدخول إلى النادي.

### 2. من هم الأعضاء؟

نحتاج طريقة لتتبع من المسموح له استخدام البنك الصغير.

```
address[] members;
mapping(address => bool) public registeredMembers;
```

إليك سبب استخدامنا لكليهما:

- members: قائمة بسيطة (array) بكل من في المجموعة.
- registeredMembers: mapping يسمح للعقد بالتحقق بسرعة إذا كان العنوان مسموحاً له بالتفاعل. (تذكّر: mappings أسرع وأرخص عند فحص الوصول!)

فكر في members كقائمة دردشة المجموعة. وregisteredMembers مثل بوابة الأمان التي تقول نعم أو لا عندما يحاول شخص ما التفاعل مع العقد.

### 3. كم حفظ كل شخص؟

بالطبع، نحتاج أن نعرف كم أودع كل عضو مع مرور الزمن.

```
mapping(address => uint256) balance;
```

هذا هو قلب البنك الصغير. يخزن الرصيد لكل عضو.

## إعداد الأمور: constructor

الآن بعد أن عرفنا ما نحتاجه، دعنا نكتب الجزء الذي يضع كل شيء في الحركة.

```
constructor() {
    bankManager = msg.sender;
    members.push(msg.sender);
}
```

إليك ما يحدث:

- نعيّن bankManager للشخص الذي نشر العقد.
- نضيف المدير كأول عضو في البنك الصغير.

لقد فتحنا بنكنا الصغير رسمياً.

## الآن لبناء القواعد (Modifiers)

قبل أن نبدأ بكتابة الميزات الفعلية، نحتاج أن نتحدث عن من يمكنه القيام بماذا.

هنا تأتي الـ modifiers — قطع صغيرة من المنطق القابل لإعادة الاستخدام التي تحمي الدوال.

onlyBankManager

```
modifier onlyBankManager() {
    require(msg.sender == bankManager, "Only bank manager can perform this action");
    _;
}
```

هذا الـ modifier يضمن أن المدير فقط يمكنه استدعاء دوال معينة — مثل إضافة الأعضاء.

إذا حاول شخص آخر؟ العقد يقول: "لا. الوصول مرفوض."

### onlyRegisteredMember

```
modifier onlyRegisteredMember() {
    require(registeredMembers[msg.sender], "Member not registered");
    _;
}
```

هذا يضمن أن فقط الأشخاص الذين تمت إضافتهم رسمياً إلى النادي يمكنهم إيداع أو سحب المدخرات.

حسناً — الآن بعد أن وضعنا الأدوار والذاكرة، دعنا ننتقل إلى الميزات الفعلية.

## إضافة أعضاء جدد

لنفترض أن المدير يريد إضافة أحد أصدقائه، Alex.

هذه هي الدالة:

```
function addMembers(address _member) public onlyBankManager {
    require(_member != address(0), "Invalid address");
    require(_member != msg.sender, "Bank Manager is already a member");
    require(!registeredMembers[_member], "Member already registered");

    registeredMembers[_member] = true;
    members.push(_member);
}
```

نفحص:

- هل العنوان صالح؟
- هل الشخص بالفعل عضو؟
- هل المدير يحاول إضافة نفسه مرة أخرى؟

عندما يبدو كل شيء جيداً، نوافق على العضو الجديد.

### عرض الأعضاء

لنفترض أن أحدهم يريد رؤية من هم جميعاً في المجموعة. ربما ليتأكد أنه تمت إضافته.

```
function getMembers() public view returns (address[] memory) {
    return members;
}
```

هذه دالة view عامة — ببساطة تُعيد قائمة الأعضاء كاملة.

## الإيداع (مدخرات محاكاة)

الآن نصل إلى الجزء الممتع.

لنفترض أنك تريد محاكاة إضافة 100 وحدة إلى بنكك الصغير.

هذه هي الدالة:

```
function deposit(uint256 _amount) public onlyRegisteredMember {
    require(_amount > 0, "Invalid amount");
    balance[msg.sender] += _amount;
}
```

هذه الدالة:

- تتحقق أنك عضو مسجل.
- تزيد رصيدك بالمبلغ الذي حددته.

في هذه المرحلة، لا نتعامل بعد مع ETH حقيقي. هذه مجرد أرقام — تمثيل للمدخرات.

فكر في هذا كنموذج أولي. تختبر النظام قبل وضع الأموال الحقيقية.

## سحب المدخرات (على الورق)

لقد ادخرت 100 وحدة. الآن تريد سحب 30.

```
function withdraw(uint256 _amount) public onlyRegisteredMember {
    require(_amount > 0, "Invalid amount");
    require(balance[msg.sender] >= _amount, "Insufficient balance");
    balance[msg.sender] -= _amount;
}
```

نفحص:

- هل أنت عضو؟
- هل لديك ما يكفي من المدخرات لسحب هذا المبلغ؟

لا يتم تحويل Ether. هذه مجرد تحديث داخلي — ما زلنا في وضع المحاكاة.

## لكن ماذا لو أردنا إيداع ETH حقيقي؟

حتى الآن، كنا نلعب بالأرقام فقط.

لكن الآن يقول أصدقاؤك:

"هذا رائع. لكن ماذا لو بدأنا حقاً في توفير Ether حقيقي؟ مثل... إرسال المال فعلاً إلى البنك الصغير؟"

هنا نقدم مفهومين جديدين:

- `payable`: كلمة مفتاحية تخبر الدالة أنها يمكن أن تستقبل ETH حقيقي.
- `msg.value`: متغير خاص يخبر العقد بالضبط كم من ETH أُرسل مع المعاملة.

## إيداع ETH حقيقي في البنك الصغير

لنكتب نسخة جديدة من دالة الإيداع — واحدة تقبل ETH حقيقي.

```
function depositAmountEther() public payable onlyRegisteredMember {
    require(msg.value > 0, "Invalid amount");
    balance[msg.sender] += msg.value;
}
```

إليك ما يحدث:

- أضفنا الكلمة المفتاحية `payable` إلى الدالة.
- لم نعد نطلب مبلغاً كمدخل؛ بل نستخدم `msg.value`.

فعندما ينادي عضو هذه الدالة ويضمن Ether في المعاملة:

- يذهب الـ ETH مباشرة إلى رصيد العقد.
- نحدّث خريطة الرصيد الشخصية حتى يحصل المستخدم على الائتمان.

هذه مدخرات حقيقية الآن — ETH فعلي، متعقب لكل مستخدم، مثل البنك.

## ماذا بنينا؟

- نظام آمن يُدار بواسطة مشرف.
- قواعد عضوية خاصة بالمجموعة.
- تتبع داخلي للأرصدة الفردية.
- القدرة على استقبال وتتبع Ether الحقيقي على السلسلة.

من المنطق البسيط إلى إدارة ETH الفعلية، لقد دخلت الآن عالم العقود الذكية الحقيقية.

## إلى أين يمكن أن نتجه من هنا؟

يمكنك الآن:

- إضافة دالة سحب تُرسل ETH فعلياً إلى محفظة العضو.
- إضافة نظام لدفع فائدة، أو مشاركة مكافأة جماعية.

قد يكون هذا البنك الصغير قد بدأ بك وبأصدقائك فقط...

لكن الآن؟ أصبح نظاماً شرعياً على السلسلة — وقد بنيته أنت.

لنستمر.

---

## فهم الدوال القابلة للدفع (Payable)

### ما هو `payable`؟

```solidity
function deposit() public payable {
    // Can receive ETH
}

function withdraw() public {
    // Cannot receive ETH
}
```

**الكلمة المفتاحية `payable`** تخبر Solidity: "هذه الدالة يمكنها استلام Ether."

**بدون `payable`:** تُعاد المعاملة إذا أرسلت ETH.

### فهم `msg.value`

```solidity
function depositAmountEther() public payable {
    require(msg.value > 0, "Must send ETH");
    balance[msg.sender] += msg.value;
}
```

**`msg.value`** هو متغير خاص يحتوي على كمية الـ wei المرسلة مع المعاملة.

**مثال:**

- المستخدم يرسل 1 ETH
- `msg.value = 1000000000000000000` (1 ETH بالـ wei)
- العقد يمنح رصيد المستخدم

---

## سحب ETH الحقيقي

### دالة السحب

```solidity
function withdrawEther(uint256 _amount) public onlyRegisteredMember {
    require(_amount > 0, "Invalid amount");
    require(balance[msg.sender] >= _amount, "Insufficient balance");

    balance[msg.sender] -= _amount;

    (bool success, ) = payable(msg.sender).call{value: _amount}("");
    require(success, "Transfer failed");
}
```

**النقاط الرئيسية:**

1. **افحص الرصيد أولاً** - تأكد أن المستخدم لديه ما يكفي
2. **حدّث الحالة قبل التحويل** - نمط Checks-Effects-Interactions
3. **استخدم `call()` للتحويلات** - الممارسة الحديثة الأفضل
4. **تحقق من النجاح** - ألغِ إذا فشل التحويل

### لماذا Checks-Effects-Interactions؟

```solidity
// VULNERABLE (Effects after Interactions)
function badWithdraw(uint256 _amount) public {
    (bool success, ) = payable(msg.sender).call{value: _amount}("");
    balance[msg.sender] -= _amount; // Too late!
}

// SAFE (Effects before Interactions)
function goodWithdraw(uint256 _amount) public {
    balance[msg.sender] -= _amount; // Update first
    (bool success, ) = payable(msg.sender).call{value: _amount}("");
}
```

**لماذا؟** يمنع هجمات إعادة الدخول حيث تعيد عقود خبيثة الاتصال قبل تحديث الحالة.

---

## رصيد العقد مقابل أرصدة المستخدمين

### نوعان من الأرصدة

```solidity
// Individual user balances (in storage)
mapping(address => uint256) balance;

// Total contract balance (actual ETH held)
address(this).balance
```

**مثال:**

- Alice تودع 1 ETH → `balance[Alice] = 1 ETH`، العقد يحتفظ بـ 1 ETH
- Bob يودع 2 ETH → `balance[Bob] = 2 ETH`، العقد يحتفظ بـ 3 ETH
- رصيد العقد: 3 ETH إجمالاً
- التتبع الفردي: Alice (1 ETH)، Bob (2 ETH)

**لماذا نتتبع الاثنين؟**

- رصيد العقد: إجمالي ETH المحتفظ به
- أرصدة المستخدمين: من يملك ماذا

---

## المفاهيم الأساسية التي تعلمتها

**1. الدوال القابلة للدفع (payable)** - استقبال ETH في العقود

**2. msg.value** - كمية ETH المرسلة مع المعاملة

**3. تحويلات ETH** - استخدام `call()` لإرسال ETH

**4. Checks-Effects-Interactions** - نمط أمني للسحوبات

**5. تتبع الأرصدة** - رصيد العقد مقابل أرصدة المستخدمين

**6. التحكم في الوصول** - من يمكنه الإيداع/السحب

---

## التطبيقات الحقيقية

### بروتوكولات DeFi

**Aave, Compound, MakerDAO:**

- المستخدمون يودعون ETH
- يحصلون على فائدة مع مرور الوقت
- يسحبون متى أرادوا

**نفس النمط مثل بنك التوفير لدينا!**

### قسّام الدفعات

```solidity
// Revenue sharing contract
function withdraw() public {
    uint256 share = balance[msg.sender];
    balance[msg.sender] = 0;
    payable(msg.sender).call{value: share}("");
}
```

### صناديق التوفير

**توفير جماعي لهدف:**

- الأصدقاء يجمعون المال
- يُقفل حتى يُحقّق الهدف
- يُوزع بعدالة

---

## اعتبارات أمانية

### 1. حماية من إعادة الدخول (Reentrancy)

**أضف ReentrancyGuard:**

```solidity
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

contract PiggyBank is ReentrancyGuard {
    function withdrawEther(uint256 _amount) public nonReentrant {
        // Safe from reentrancy
    }
}
```

### 2. تجاوز الأعداد (Integer Overflow)

**Solidity 0.8+** لديها حماية تلقائية، لكن كن حذراً:

```solidity
balance[msg.sender] += msg.value; // Safe in 0.8+
```

### 3. التحويلات الفاشلة

**تحقق دائماً من النجاح:**

```solidity
(bool success, ) = payable(msg.sender).call{value: _amount}("");
require(success, "Transfer failed");
```

---

## لماذا هذا مهم

التعامل مع ETH الحقيقي هو أساس:

- **DeFi** - أكثر من 100 مليار دولار في البروتوكولات
- **المدفوعات** - التحويلات عبر الحدود
- **التوفير** - حسابات بعوائد
- **الاستثمار** - صناديق مشتركة
- **الألعاب** - اقتصادات داخلية

كل بروتوكول رئيسي يتعامل مع ETH بهذه الطريقة.

## تحدَّ نفسك

وسع هذا البنك الصغير:

1. أضف حساب فائدة للمدخرين على المدى الطويل
2. طبّق حدود سحب (يومي/أسبوعي)
3. أنشئ هدفاً جماعياً مع أموال مقفلة
4. أضف سحب طارئ مع عقوبة
5. ابنِ ميزة توفير مقفل زمنياً

لقد أتقنت التعامل مع ETH الحقيقي. هذا تطوير عقود ذكية جاهز للإنتاج!

</div>
