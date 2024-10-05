---
layout: page
title: "Generating Test Codes with AI"
description: Towards a best practice of building research code
img: assets/img/etc/test_cases.png
importance: 1
category: Tips
related_publications: false
date: 2024-10-04 16:40:16
giscus_comments: true
---


*Written by Bumjin on 24.10.05*

<div class="row justify-content-sm-center ">
    <div class="col-sm-5 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/etc/test_cases.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    우리가 생각하는 Success Case들을 더욱 효율적으로 만들 수 있어. 
</div>



<br>
<br>


## Intro 

보통 작성된 코드는 의도대로 동작하지 않는 경우가 많다. 구현에 사용되는 모든 라이브러리의 작동 원리를 완전히 이해하고 이를 올바르게 사용하는 것은 결코 쉬운 일이 아니다. 수많은 시행착오 끝에 더 깔끔하고 제대로 동작하는 코드를 작성할 수는 있지만, 그런 경우는 전체의 10%도 채 되지 않는 것 같다. 특히, 실험 코드를 처음 작성하는 경우라면 그 성공 확률은 0%에 가까울 것이다.

또 다른 문제는 테스트에서 검증이 완료되면 이후 발생할 수 있는 문제를 충분히 고려하지 않는다는 점이다. 이러한 편협한 시각은 물론 제한된 시간에서 기인하는 것이므로 이를 개인의 부주의나 책임으로만 돌릴 수는 없다.

AI 시대에 딥러닝 실험 코드를 개발하는 과정에서 가장 필요한 것은 테스트 코드를 자동으로 생성해주는 도구나 샘플들이다. 즉, 함수가 의도한 대로 동작하는지 확인하고, 입출력에 대한 기대값을 정확하게 반영할 수 있는 샘플들을 AI가 생성해야 한다는 것이다. 나는 AI가 생성한 수많은 테스트 코드, 어서션(assertion), 그리고 측적(measure) 등이 더 확실한 코드를 만들어 줄 것이라 믿어 의심치 않는다.

**Generating** 

1. Samples for a function 
2. Saving outputs 
3. Load and Visualizing outputs. 
4. When using random functions


### 1. Samples for a function 

다음과 같이 Text를 normalize해주는 함수를 고려해보자. 

```python
def normalize_text(s):
    """Removing articles and punctuation, and standardizing whitespace are all typical text processing steps."""
    import string, re

    def remove_articles(text):
        regex = re.compile(r"\b(a|an|the)\b", re.UNICODE)
        return re.sub(regex, " ", text)

    def white_space_fix(text):
        return " ".join(text.split())

    def remove_punc(text):
        exclude = set(string.punctuation)
        return "".join(ch for ch in text if ch not in exclude)

    def lower(text):
        return text.lower()

    return white_space_fix(remove_articles(remove_punc(lower(s))))

```

해당 코드에 대한 입출력 예시들을 AI로부터 자동으로 만들 수 있다. 


<div class="row justify-content-sm-center ">
    <div class="col-sm-8 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/etc/test_cases_example.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    ChatGPT-4o 로 만들 Example들 
</div>


이러한 방식의 장점은 다음과 같다. 

1. 함수에 입력으로 들어가는 다양한 경우를 가정해줄 수 있다. (우리의 인식을 넘어서, `Hello` -> `hello`)
2. 입력에 대한 다양한 경우들을 세분화 해서 제공할 수 있다. 보다 효율적인 AI 활용 방법은 


다음 경우들을 보자.

1. **기사 제거 (Article Removal)**  
   "An apple a day keeps the doctor away." -> "apple day keeps doctor away"

2. **구두점 제거 (Punctuation Removal)**  
   "Hello, world!" -> "hello world"

3. **여러 공백 제거 (Whitespace Normalization)**  
   " This sentence has extra spaces. " -> "this sentence has extra spaces"

4. **대소문자 변환 (Lowercase Conversion)**  
   "THE QUICK BROWN FOX" -> "quick brown fox"

5. **숫자와 구두점 포함 (Numbers and Punctuation)**  
   "The year 2024 will be amazing." -> "year 2024 will be amazing"

6. **혼합된 구두점 및 공백 (Mixed Punctuation and Whitespace)**  
   "Hello... world? An example!" -> "hello world example"

7. **공백으로만 이루어진 입력 (All Whitespace Input)**  
   " A man with a plan." -> "man with plan"

8. **기사가 여러 번 반복된 경우 (Multiple Articles in a Row)**  
   "A a an the the an a" -> ""

9. **구두점만 포함된 입력 (All Punctuation Input)**  
   "!!!" -> ""

10. **빈 입력 (Empty Input)**  
   "" -> ""

---

### 2. Saving outputs

TBD

### 3. Load and Visualizing outputs. 

TBD

### 4. When using random functions

TBD