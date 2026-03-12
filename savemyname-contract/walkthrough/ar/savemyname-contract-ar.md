<div dir="rtl">

# عقد SaveMyName: فهم السلاسل النصية ومواقع البيانات

مرحبًا بك في درس أساسي في Solidity: **التعامل مع السلاسل النصية وفهم مواقع البيانات**. اليوم، ستتعلم كيف تخزن وتسترجع بيانات نصية على البلوكتشين — ولماذا الأمر أكثر تعقيدًا من تخزين الأرقام.

## لماذا السلاسل النصية مختلفة

**الأنواع البسيطة (الأرقام):**

```solidity
uint256 public age = 25;  // حجم ثابت: 32 بايت
bool public active = true; // حجم ثابت: 1 بايت
address public owner;      // حجم ثابت: 20 بايت
```

**الأنواع المعقدة (السلاسل):**

```solidity
string public name = "Alice";           // حجم متغير: 5 بايت
string public bio = "Solidity dev...";  // حجم متغير: 15+ بايت
```

**الفرق:**

- **الأرقام** - حجم ثابت وتُخزَّن مباشرة
- **السلاسل** - حجم متغير وتُخزَّن كمصفوفات ديناميكية
- **السلاسل** - أغلى في التخزين والمعالجة
- **السلاسل** - تحتاج معالجة خاصة (كلمات `memory`/`storage`)

## تكلفة تخزين السلاسل النصية

**تكلفة `Gas`:**

- تخزين `uint256`: ~20,000 `Gas`
- تخزين سلسلة قصيرة (5 حروف): ~25,000 `Gas`
- تخزين سلسلة طويلة (100 حرف): ~100,000+ `Gas`

**لماذا مكلفة؟** السلاسل تُخزَّن كمصفوفات بايت. كل حرف يكلف `Gas` للتخزين.

### تخزين اسم وسيرة ذاتية على البلوكتشين

```
string name;
string bio;
```

#### ماذا يحدث هنا؟

`string name;` و `string bio;` هما متغيران للحالة يتم تخزينهما بشكل دائم على البلوكتشين.

### دالة `add()` - تخزين البيانات

```
function add(string memory _name, string memory _bio) public {
    name = _name;
    bio = _bio;
}
```

#### تحليل الدالة

`_name` و `_bio` هما معطى الدالة. نستخدم كلمة `memory` لأنها سلاسل معقدة وتحتاج تخزينًا مؤقتًا أثناء تنفيذ الدالة.

### دالة `retrieve()` - جلب البيانات من البلوكتشين

```
function retrieve() public view returns (string memory, string memory) {
    return (name, bio);
}
```

#### فهم دوال `view`

كلمة `view` تخبر Solidity أن هذه الدالة تقرأ البيانات فقط ولا تكلف `Gas` عند استدعائها خارجيًا.

### تحسين العقد

يمكنك دمجهما في دالة واحدة، لكن قد يزيد ذلك من تكلفة `Gas` إذا غيّرت الحالة:

```
function saveAndRetrieve(string memory _name, string memory _bio) public returns (string memory, string memory) {
    name = _name;
    bio = _bio;
    return (name, bio);
}
```

---

## فهم مواقع البيانات

لدى Solidity ثلاث مواقع للبيانات: **storage**، **memory**، و **calldata**.

### `storage` - التخزين الدائم على البلوكتشين

```solidity
string public name;  // مخزن بشكل دائم على البلوكتشين
string public bio;   // مخزن بشكل دائم على البلوكتشين
```

**الخصائص:**

- يستمر بين استدعاءات الدوال
- الأغلى (يكلف `Gas` للكتابة)
- متغيرات الحالة دائمًا في `storage`
- قابل للتعديل

**مثال:**

```solidity
function updateName(string memory _name) public {
    name = _name;  // يكتب في storage (مكلف!)
}
```

### `memory` - التخزين المؤقت داخل الدالة

```solidity
function add(string memory _name, string memory _bio) public {
    // _name و _bio موجودان فقط أثناء تنفيذ الدالة
    name = _name;
    bio = _bio;
}
```

**الخصائص:**

- مؤقت، يتم مسحه بعد انتهاء الدالة
- أرخص من `storage`
- مطلوب للأنواع المعقدة في معطيات الدالة
- قابل للتعديل

**لماذا نستخدم `memory`؟**

```solidity
// هذا لن يُجمّع!
function add(string _name) public {  // ناقص 'memory'
    name = _name;
}

// هذا يعمل!
function add(string memory _name) public {
    name = _name;
}
```

### `calldata` - مدخلات الدالة للقراءة فقط

```solidity
function add(string calldata _name, string calldata _bio) external {
    // _name و _bio للقراءة فقط
    name = _name;
    bio = _bio;
}
```

**الخصائص:**

- للقراءة فقط (لا يمكن تعديلها)
- الأرخص تكلفة
- فقط لمعايير الدوال الخارجية (`external`)
- الأفضل لتحسين تكلفة `Gas`

**مقارنة:**

| الموقع       | قابل للتعديل | الاستمرارية | التكلفة | حالة الاستخدام |
| ------------ | ------------ | ----------- | ------- | -------------- |
| **storage**  | نعم          | دائم        | عالي    | متغيرات الحالة |
| **memory**   | نعم          | مؤقت        | متوسط   | متغيرات الدالة |
| **calldata** | لا           | مؤقت        | منخفض   | مدخلات خارجية  |

---

## عمليات السلاسل النصية والقيود

### ما الذي لا يمكنك فعله مع السلاسل

```solidity
// ❌ لا يمكن الدمج مباشرةً
string result = "Hello" + "World";  // لا يعمل!

// ❌ لا يمكن المقارنة مباشرةً
if (name == "Alice") { }  // لا يعمل!

// ❌ لا يمكن الحصول على الطول بسهولة
uint len = name.length;  // لا يعمل!

// ❌ لا يمكن الوصول للحرف حسب الفهرس
string firstChar = name[0];  // لا يعمل!
```

### ما الذي يمكنك فعله

**1. التخزين والاسترجاع:**

```solidity
string public name;

function setName(string memory _name) public {
    name = _name;
}

function getName() public view returns (string memory) {
    return name;
}
```

**2. المقارنة باستخدام `keccak256`:**

```solidity
function isNameAlice(string memory _name) public pure returns (bool) {
    return keccak256(abi.encodePacked(_name)) == keccak256(abi.encodePacked("Alice"));
}
```

**3. التحويل إلى `bytes` للتعامل معها:**

```solidity
function getLength(string memory _str) public pure returns (uint) {
    return bytes(_str).length;
}
```

---

## أنماط متقدمة للسلاسل النصية

### 1. استخدام `bytes32` بدلاً من `string`

**للنصوص القصيرة وثابتة الطول:**

```solidity
// مكلف
string public name;  // حجم متغير، ديناميكي

// أرخص
bytes32 public name;  // حجم ثابت، أقصى 32 بايت

function setName(string memory _name) public {
    require(bytes(_name).length <= 32, "Name too long");
    name = bytes32(bytes(_name));
}

function getName() public view returns (string memory) {
    return string(abi.encodePacked(name));
}
```

**التوفير:** ~50% تقليل في `Gas` للنصوص القصيرة!

### 2. دمج السلاسل

```solidity
function concatenate(string memory a, string memory b) public pure returns (string memory) {
    return string(abi.encodePacked(a, b));
}

// مثال: concatenate("Hello", "World") تعيد "HelloWorld"
```

### 3. مقارنة السلاسل

```solidity
function compareStrings(string memory a, string memory b) public pure returns (bool) {
    return keccak256(abi.encodePacked(a)) == keccak256(abi.encodePacked(b));
}
```

### 4. إعادة أكثر من قيمة

```solidity
function getUserInfo() public view returns (string memory, string memory, uint256) {
    return (name, bio, block.timestamp);
}

// الاستخدام:
// (string memory userName, string memory userBio, uint256 timestamp) = getUserInfo();
```

---

## حالات استخدام واقعية

### ENS (Ethereum Name Service)

**يخزن أسماء الدومين كسلاسل نصية:**

```solidity
mapping(bytes32 => string) public names;

function setName(bytes32 node, string memory name) public {
    names[node] = name;
}
```

### بيانات الـ NFT

**يخزن عناوين URI الخاصة بالتوكن:**

```solidity
mapping(uint256 => string) private _tokenURIs;

function tokenURI(uint256 tokenId) public view returns (string memory) {
    return _tokenURIs[tokenId];
}
```

### ملفات تعريف اجتماعية

**يخزن السير الذاتية للمستخدمين:**

```solidity
struct Profile {
    string username;
    string bio;
    string avatarURL;
}

mapping(address => Profile) public profiles;
```

---

## نصائح تحسين `Gas`

### 1. استخدم الأحداث للبيانات التاريخية

```solidity
event NameChanged(address indexed user, string newName, uint256 timestamp);

function setName(string memory _name) public {
    emit NameChanged(msg.sender, _name, block.timestamp);
    // لا تخزن في الحالة إن كنت تحتاج السجل فقط
}
```

**لماذا؟** الأحداث أرخص بكثير من التخزين (~2,000 `Gas` مقابل ~20,000 `Gas`).

### 2. خزن التجزئات بدلًا من السلاسل كاملةً

```solidity
// مكلف
mapping(address => string) public documents;

// أرخص
mapping(address => bytes32) public documentHashes;

function storeDocument(string memory doc) public {
    documentHashes[msg.sender] = keccak256(abi.encodePacked(doc));
}

function verifyDocument(string memory doc) public view returns (bool) {
    return documentHashes[msg.sender] == keccak256(abi.encodePacked(doc));
}
```

### 3. استخدم IPFS للنصوص الكبيرة

```solidity
// خزن هاش IPFS (46 بايت) بدلًا من المحتوى الكامل
mapping(address => string) public ipfsHashes;

function setContent(string memory ipfsHash) public {
    ipfsHashes[msg.sender] = ipfsHash;
    // المحتوى الفعلي مخزن خارج السلسلة على IPFS
}
```

---

## الأخطاء الشائعة

### 1. نسيان موقع البيانات

```solidity
// ❌ لن يُجمّع
function setName(string _name) public {
    name = _name;
}

// ✅ الصحيح
function setName(string memory _name) public {
    name = _name;
}
```

### 2. استخدام `storage` حين تريد `memory`

```solidity
// ❌ مكلف وخطأ
function processName(string storage _name) internal {
    // يغير متغير الحالة!
}

// ✅ صحيح
function processName(string memory _name) internal {
    // يعمل على نسخة في الذاكرة فقط
}

function processName(string memory _name) internal pure {
    // يعمل على نسخة
}
```

### 3. عدم التحقق من طول السلسلة

```solidity
// ❌ قد يخزن سلاسل ضخمة (مكلف!)
function setBio(string memory _bio) public {
    bio = _bio;
}

// ✅ تحديد الطول
function setBio(string memory _bio) public {
    require(bytes(_bio).length <= 280, "السيرة طويلة جدًا");
    bio = _bio;
}
```

---

## المفاهيم الرئيسية التي تعلمتها

**1. تخزين السلاسل** - كيف تُخزَّن البيانات النصية على البلوكتشين

**2. مواقع البيانات** - الفروقات بين `storage`، `memory`، و `calldata`

**3. تكلفة `Gas`** - لماذا السلاسل مكلفة

**4. عمليات السلاسل** - المقارنة، الدمج، التحويل

**5. التحسين** - `bytes32`، الأحداث، أنماط IPFS

**6. دوال `view`** - القراءة المجانية مقابل الكتابة المكلفة

---

## لماذا هذا مهم

السلاسل موجودة في كل مكان في عالم Web3:

- **ENS** - أسماء الدومين
- **NFTs** - عناوين الميتاداتا
- **التواصل الاجتماعي** - أسماء المستخدمين، السير الذاتية
- **DeFi** - رموز التوكن، الأسماء

فهم كيف تتعامل مع السلاسل بكفاءة أمر أساسي لبناء تطبيقات حقيقية.

## تحدى نفسك

قم بتوسيع هذا العقد:

1. أضف تحققًا لطول السلسلة
2. نفّذ دمجًا للسلاسل لبناء اسم كامل
3. أنشئ تحققًا من تميّز أسماء المستخدمين
4. ابنِ نظام ملف تعريفي مع عدة حقول نصية
5. أضف تكامل IPFS للمحتوى النصي الكبير

لقد أتقنت السلاسل ومواقع البيانات. هذه معرفة أساسية في Solidity!

<div/>