<div dir="rtl">


# آموزش Rule در لاراول ۱۲

## Validation Rules در لاراول ۱۲

در لاراول ۱۲ می‌توانید با متد `validate` اعتبارسنجی درخواست‌ها را انجام دهید. قوانین اعتبارسنجی به صورت رشته یا آرایه تعریف می‌شوند:
<div dir="ltr">
```
$request->validate([
    'name' => 'required|string|max:255',
    'email' => 'required|email|unique:users',
    'password' => 'required|min:8|confirmed',
]);
```
</div>
مثلا برای ایمیل از قوانین `required` و `email` و `unique` استفاده می‌شود [translate:لاراول ۱۲].  


## شرطی کردن Validation بر اساس سایر فیلدها

می‌توانید یک فیلد را منوط به پر شدن یا مقدار فیلد دیگر کنید. به عنوان مثال `password` منوط به پر بودن `email`:
<div dir="ltr">
```
$request->validate([
    'email' => 'required|email',
    'password' => 'required_with:email|min:8',
]);
```
</div>

یا با متدهای شرطی پیچیده‌تر با `Rule::when`:

<div dir="ltr">
  
```
use Illuminate\Validation\Rule;

$request->validate([
    'email' => 'required|email',
    'password' => [
        'required',
        Rule::when($request->filled('email') && filter_var($request->email, FILTER_VALIDATE_EMAIL), [
            'confirmed',
            'regex:/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).+$/'
        ])
    ],
]);
```
</div>
## ساخت Rule سفارشی در لاراول ۱۲

### ایجاد کلاس Rule

با دستور زیر یک کلاس rule سفارشی بسازید:

<div dir="ltr">
  
```
php artisan make:rule BirthYearRule
```

</div>

در کلاس ساخته‌شده می‌توانید منطق اعتبارسنجی خود را تعریف کنید:

<div dir="ltr">
  
```
<?php

namespace App\Rules;

use Closure;
use Illuminate\Contracts\Validation\ValidationRule;

class BirthYearRule implements ValidationRule
{
    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        if (!($value > 1980 && $value < date('Y'))) {
            $fail('فیلد :attribute باید بین ۱۹۸۰ تا سال جاری باشد.');
        }
    }
}
```

### استفاده در کنترلر

```
use App\Rules\BirthYearRule;

$request->validate([
    'birth_year' => ['required', new BirthYearRule()],
]);
```

</div>

### Rule با پارامتر

مثال rule دریافت‌کننده پارامتر:

<div dir="ltr">
  
```
class MinimumAgeRule implements ValidationRule
{
    protected $minimumAge;

    public function __construct($minimumAge)
    {
        $this->minimumAge = $minimumAge;
    }

    public function validate(string $attribute, mixed $value, Closure $fail): void
    {
        $age = date('Y') - (int) $value;
        if ($age < $this->minimumAge) {
            $fail("فیلد :attribute باید حداقل {$this->minimumAge} سال باشد.");
        }
    }
}
```

استفاده:

```
$request->validate([
    'birth_year' => ['required', new MinimumAgeRule(18)],
]);
```

</div>

### Closure برای Ruleهای سریع

<div dir="ltr">
  
```
$request->validate([
    'username' => [
        'required',
        function ($attribute, $value, $fail) {
            if (!preg_match('/^[a-zA-Z0-9]+$/', $value)) {
                $fail('فیلد :attribute فقط باید شامل حروف و اعداد باشد.');
            }
        },
    ],
]);
```
</div>

## `bail` در validation

[translate:bail] یک قانون اعتبارسنجی است که پس از اولین خطا برای یک فیلد دیگر rules آن فیلد را بررسی نمی‌کند و به validation سایر فیلدها ادامه می‌دهد.

مثال:

<div dir="ltr">
  
```
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'content' => 'required',
]);
```

</div>


اگر `title` خالی باشد، بعد از `required` قوانین `unique` و `max` بررسی نمی‌شوند، اما `content` بررسی می‌شود.

### مزایای bail

- نمایش یک خطای مرتبط برای هر فیلد  
- بهبود عملکرد با جلوگیری از اجرای قوانین غیرضروری

## تفاوت bail با stopOnFirstFailure

| ویژگی          | bail                                   | stopOnFirstFailure                       |
|----------------|--------------------------------------|----------------------------------------|
| سطح اثرگذاری  | فقط روی یک فیلد خاص                   | روی کل فرآیند validation               |
| توقف بعد از خطا | بقیه rules همان فیلد بررسی نمی‌شوند  | بقیه فیلدها بررسی نمی‌شوند             |
| نوع            | قانون validation                     | ویژگی یا متد Validator                  |
| استفاده        | `'field' => 'bail|required|email'`    | `protected $stopOnFirstFailure = true;`|
| نمایش خطاها    | چند خطا از چند فیلد نمایش داده می‌شود| فقط خطای اولین فیلد نمایش داده می‌شود |

### مثال stopOnFirstFailure در FormRequest

<div dir="ltr">
  
```
protected $stopOnFirstFailure = true;

public function rules()
{
    return [
        'title' => 'bail|required|unique:posts|max:255',
        'content' => 'bail|required|min:10',
    ];
}
```

</div>


## نکات تکمیلی درباره Ruleها

- می‌توانید rules را ترکیب کنید مثل `['bail', 'required', new BirthYearRule()]`
- پیام خطا می‌تواند شامل `:attribute` باشد که نام فیلد جایگزین می‌شود
- Ruleها در ترتیب نوشتن اجرا می‌شوند
- Rule سفارشی می‌تواند دسترسی به request داشته باشد تا اعتبارسنجی پیچیده‌تر انجام شود

</div>
