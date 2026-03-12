# عقد محطة التصويت (PollStation Contract): إتقان المصفوفات والخرائط

أهلاً وسهلاً في درس أساسي في بنى البيانات في Solidity: **المصفوفات والخرائط**. اليوم، ستبني نظام تصويت يوضح كيفية تخزين وإدارة مجموعات البيانات على السلسلة.

## لماذا تهم بنى البيانات؟

**التحدي:** تحتاج إلى تخزين عدة أجزاء من البيانات ذات الصلة:

- قائمة المرشحين
- عدد الأصوات لكل مرشح
- من صوّت
- سجل التصويت

**بدون بنى بيانات مناسبة:**

```solidity
// هذا لا يتسع!
string public candidate1;
string public candidate2;
string public candidate3;
uint256 public votes1;
uint256 public votes2;
uint256 public votes3;
```

**المشاكل:**

- عدد ثابت من المرشحين
- لا يمكن المرور على المرشحين
- كود مكرر في كل مكان
- من المستحيل الحفاظ عليه

**الحل:** المصفوفات والخرائط!

### إعلان قائمة المرشحين وتتبع الأصوات

```
string[] public candidateNames;
mapping(string => uint256) voteCount;
```

#### المصفوفات – تخزين قائمة المرشحين

`string[]` هي مصفوفة من السلاسل النصية. تخزن قيماً متعددة في قائمة مرتبة.

#### الخرائط – تخزين عدد الأصوات

`mapping(string => uint256)` يربط اسم المرشح بعدد أصواته للبحث الفوري.

### إضافة المرشحين – دالة `addCandidateNames()`

```
function addCandidateNames(string memory _candidateNames) public {
    candidateNames.push(_candidateNames);
    voteCount[_candidateNames] = 0;
}
```

### استرجاع قائمة المرشحين

```
function getcandidateNames() public view returns (string[] memory) {
    return candidateNames;
}
```

### التصويت لمرشح – دالة `vote()`

```
function vote(string memory _candidateNames) public {
    voteCount[_candidateNames] += 1;
}
```

### التحقق من أصوات المرشح

```
function getVote(string memory _candidateNames) public view returns (uint256) {
    return voteCount[_candidateNames];
}
```

---

## المصفوفات مقابل الخرائط: متى تستخدم كل واحدة

### المصفوفات - القوائم المرتبة

**استخدمها عندما:**

- تحتاج إلى المرور على العناصر
- الترتيب مهم
- تحتاج إلى معرفة الطول
- تريد الوصول عن طريق الفهرس

**حالات الاستخدام:**

- قائمة المرشحين
- سجل المعاملات
- لوحات الترتيب
- قوائم حاملي الرموز

**القيود:**

- تكلفة البحث باهظة (O(n))
- تكلفة باهظة للحذف من الوسط
- تزداد تكاليف `Gas` مع الحجم

### الخرائط - أزواج المفتاح والقيمة

**استخدمها عندما:**

- تحتاج إلى عمليات بحث سريعة (O(1))
- لديك مفتاح فريد
- الترتيب غير مهم
- لا تحتاج إلى التكرار

**حالات الاستخدام:**

- عدد الأصوات حسب المرشح
- الأرصدة حسب العنوان
- سجلات الملكية
- قوائم التحكم في الوصول

**القيود:**

- لا يمكن التكرار على المفاتيح
- لا يمكن الحصول على الطول
- جميع المفاتيح موجودة (إرجاع القيمة الافتراضية)

---

## الاعتبارات الأمنية

### 1. التصويت المتكرر

**المشكلة:** لا شيء يوقف شخص ما من التصويت عدة مرات!

```solidity
// عُرضة للخطر
function vote(string memory _candidateNames) public {
    voteCount[_candidateNames] += 1;
}
```

**الحل:**

```solidity
mapping(address => bool) public hasVoted;

function vote(string memory _candidateNames) public {
    require(!hasVoted[msg.sender], "لقد صوتت بالفعل");
    hasVoted[msg.sender] = true;
    voteCount[_candidateNames] += 1;
}
```

### 2. مرشحون غير صالحين

**المشكلة:** يمكنك التصويت لمرشحين غير موجودين!

```solidity
// عُرضة للخطر
vote("RandomPerson"); // ينشئ عدد أصوات لمرشح غير موجود
```

**الحل:**

```solidity
mapping(string => bool) public isCandidate;

function addCandidateNames(string memory _candidateNames) public {
    candidateNames.push(_candidateNames);
    isCandidate[_candidateNames] = true;
    voteCount[_candidateNames] = 0;
}

function vote(string memory _candidateNames) public {
    require(isCandidate[_candidateNames], "مرشح غير صالح");
    require(!hasVoted[msg.sender], "لقد صوتت بالفعل");
    hasVoted[msg.sender] = true;
    voteCount[_candidateNames] += 1;
}
```

### 3. إضافة مرشحين بدون تفويض

**المشكلة:** أي شخص يمكنه إضافة مرشحين!

**الحل:**

```solidity
address public admin;

constructor() {
    admin = msg.sender;
}

modifier onlyAdmin() {
    require(msg.sender == admin, "ليس مسؤول");
    _;
}

function addCandidateNames(string memory _candidateNames) public onlyAdmin {
    candidateNames.push(_candidateNames);
    isCandidate[_candidateNames] = true;
    voteCount[_candidateNames] = 0;
}
```

## ميزات التصويت المتقدمة

---

## ميزات التصويت المتقدمة

### 1. فترة التصويت

```solidity
uint256 public votingStart;
uint256 public votingEnd;

constructor(uint256 _durationInDays) {
    votingStart = block.timestamp;
    votingEnd = block.timestamp + (_durationInDays * 1 days);
}

function vote(string memory _candidateNames) public {
    require(block.timestamp >= votingStart, "لم يبدأ التصويت");
    require(block.timestamp <= votingEnd, "انتهى التصويت");
    // ... باقي المنطق
}
```

### 2. الحصول على الفائز

```solidity
function getWinner() public view returns (string memory winner, uint256 winningVoteCount) {
    require(block.timestamp > votingEnd, "التصويت لا يزال نشطاً");

    winningVoteCount = 0;
    for (uint i = 0; i < candidateNames.length; i++) {
        uint256 votes = voteCount[candidateNames[i]];
        if (votes > winningVoteCount) {
            winningVoteCount = votes;
            winner = candidateNames[i];
        }
    }
}
```

### 3. سجل التصويت

```solidity
mapping(address => string) public voterChoice;

function vote(string memory _candidateNames) public {
    // ... التحقق ...
    voterChoice[msg.sender] = _candidateNames;
    voteCount[_candidateNames] += 1;
}
```

---

## نصائح تحسين `Gas`

### 1. استخدم `Bytes32` بدلاً من `String`

```solidity
// مكلف
string[] public candidateNames;
mapping(string => uint256) voteCount;

// أرخص
bytes32[] public candidateNames;
mapping(bytes32 => uint256) voteCount;
```

**لماذا؟** السلاسل النصية ديناميكية ومكلفة. `bytes32` بحجم ثابت وأرخص.

### 2. حفظ طول المصفوفة

```solidity
// مكلف - يقرأ الطول في كل تكرار
for (uint i = 0; i < candidateNames.length; i++) {
    // ...
}

// أرخص - يقرأ الطول مرة واحدة
uint256 length = candidateNames.length;
for (uint i = 0; i < length; i++) {
    // ...
}
```

### 3. استخدم `Events` للسجل

```solidity
event Voted(address indexed voter, string candidate, uint256 timestamp);

function vote(string memory _candidateNames) public {
    // ... المنطق ...
    emit Voted(msg.sender, _candidateNames, block.timestamp);
}
```

**لماذا؟** `Events` أرخص بكثير من التخزين للبيانات التاريخية.

---

## أنظمة التصويت في الحياة الحقيقية

### `Snapshot` (التصويت خارج السلسلة)

- يتم تخزين الأصوات خارج السلسلة (`IPFS`)
- التصويت بدون تكاليف `Gas`
- التنفيذ على السلسلة فقط

### `Compound Governance`

- التصويت المرجح بالرموز
- دعم التفويض
- تنفيذ `Timelock`

### `Aragon`

- تطبيقات تصويت معيارية
- استراتيجيات تصويت متعددة
- إطار عمل `DAO`

---

## المفاهيم الرئيسية التي تعلمتها

**1. المصفوفات** - قوائم مرتبة قابلة للتكرار

**2. الخرائط** - عمليات بحث سريعة عن مفتاح-قيمة

**3. اختيار بنية البيانات** - متى تستخدم كل واحدة

**4. أنماط الأمان** - منع التصويت المتكرر والتحقق

**5. تحسين `Gas`** - `bytes32` وحفظ العناصر و`Events`

**6. ميكانيكا التصويت** - الفترات والفائزون والسجل

---

## لماذا هذا مهم

المصفوفات والخرائط هي أساس كل عقد ذكي:

- **أرصدة الرموز** - `mapping(address => uint256)`
- **ملكية `NFT`** - `mapping(uint256 => address)`
- **قائمة المسموح** - `mapping(address => bool)`
- **سجل المعاملات** - مصفوفة من `structs`

إتقان هذه البنى البيانية ضروري لتطوير `Solidity`.

## تحدِّ نفسك

وسعّ نظام التصويت هذا:

1. أضف التصويت المرجح (القائم على الرموز)
2. طبّق التصويت متعدد الاختيارات المرتبة
3. أنشئ نظام استطلاعات متعددة
4. أضف تفويض الأصوات
5. ابنِ آلية التصويت التربيعي

لقد أتقنت بنى البيانات في `Solidity`. هذه معرفة أساسية!
