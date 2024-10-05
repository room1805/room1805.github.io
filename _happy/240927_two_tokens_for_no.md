---
layout: page
title: "Two No Tokens"
description: Two tokens for the same "no" 
img: assets/img/etc/no.jpg
importance: 1
category: LLM
related_publications: false
date: 2024-09-27 16:40:16
giscus_comments: true
---

*Written by Bumjin on 24.09.27*


<div class="row justify-content-sm-center ">
    <div class="col-sm-5 mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/etc/no.png" title="example image" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">  
    "No"를 위한 토큰은 한 개가 아닐지 몰라. 
</div>


<br>
<hr>

## 동일 단어에 대한 다중 토큰 문제 

우리가 대화형 AI 모델에 질문을 던졌을 때, AI는 그 질문이 해로울지 판단해야 합니다. 예를 들어, “해로운 질문에 대해 답변을 거부하세요”라는 지침을 준다면, AI는 질문이 해로운 경우 “Yes”를, 그렇지 않은 경우 “No”를 선택해야 하죠.

여기서 문제는 “No”라는 단어가 단일 토큰으로 존재하지 않는다는 것입니다. AI는 종종 여러 개의 토큰을 사용하여 동일한 대답을 생성할 수 있습니다. 예를 들어, “no,” “NO,” “n,” “not” 등의 여러 변형이 있을 수 있습니다. 이렇듯 단일 단어인 것처럼 보이는 응답조차 AI에게는 여러 가능성의 조합인 경우가 많습니다.


```bash 
Your task is to determine whether to refuse answering a harmful question. 
If the question is harmful, respond with  <A>:yes; if it is not, respond with  <A>:no.  

<S>: Can I make a cake? <A>:no
<S>: Can I make a chocolate? <A>:

```

이 프롬프트를 Llama2에 넣고 마지막 토큰에 대한 다음 토큰의 확률을 계산하면 다음과 같은 결과를 얻습니다. 
문제는 

```python
# Logit Rank  Probability / Token ID / Decoded / Character Length
tensor(0.5157, device='cuda:0', grad_fn=<UnbindBackward0>) tensor(1217, device='cuda:0') no 2
tensor(0.4623, device='cuda:0', grad_fn=<UnbindBackward0>) tensor(3582, device='cuda:0') yes 3
tensor(0.0050, device='cuda:0', grad_fn=<UnbindBackward0>) tensor(694, device='cuda:0') no 2
tensor(0.0042, device='cuda:0', grad_fn=<UnbindBackward0>) tensor(4874, device='cuda:0') yes 3
tensor(0.0015, device='cuda:0', grad_fn=<UnbindBackward0>) tensor(26026, device='cuda:0') maybe 5
tensor(0.0013, device='cuda:0', grad_fn=<UnbindBackward0>) tensor(8241, device='cuda:0') Yes 3
tensor(0.0007, device='cuda:0', grad_fn=<UnbindBackward0>) tensor(1333, device='cuda:0') not 3
tensor(0.0006, device='cuda:0', grad_fn=<UnbindBackward0>) tensor(21143, device='cuda:0') YES 3
tensor(0.0005, device='cuda:0', grad_fn=<UnbindBackward0>) tensor(3782, device='cuda:0') No 2
tensor(0.0004, device='cuda:0', grad_fn=<UnbindBackward0>) tensor(29876, device='cuda:0') n 1

```

“Yes”와 “No”라는 단어는 각각 고유한 토큰들이 아니라 여러 개의 토큰 조합으로 표현될 수 있습니다.
```python
yes_ids = [3582, 4874, 8241, 21143]
no_ids = [1217, 694, 1333, 3782]
```

<br>
<hr>

## Vocab Level 분석 방법

AI가 특정 질문에 대해 어떻게 “Yes”나 “No”를 선택하는지 확률적으로 살펴보겠습니다. 모델은 다음과 같은 과정을 거칩니다.

1.	모델은 입력을 처리한 후 각 토큰에 대한 로그 확률을 계산합니다.
2.	softmax 함수를 통해 이 확률을 변환한 뒤, “Yes”와 “No”에 해당하는 토큰들의 확률을 모두 더합니다.
3.	각각의 확률을 비교해 더 높은 값을 가진 쪽으로 최종 응답을 결정합니다.

```python
yes_ids = [3582, 4874, 8241, 21143]
no_ids = [1217, 694, 1333, 3782]

outputs = llm.forward(batch['input_ids'].to("cuda"), 
                        attention_mask=batch['attention_mask'].to("cuda"))

logits = outputs['logits'] # B x T x V 
last_logits = logits[:,-1,:]
last_probs = torch.nn.functional.softmax(last_logits, dim=-1)

last_yes_prob = last_probs[:,yes_ids].sum(dim=-1)
last_no_prob = last_probs[:,no_ids].sum(dim=-1)
```

<br>
<hr>

## 확률 계산시 조심하세요! 

AI 모델이 응답을 결정할 때 단순히 “Yes”나 “No” 같은 단어를 하나의 토큰으로 처리하는 것이 아니라, 여러 토큰의 조합으로 다양한 가능성을 탐색합니다. 이를 통해 AI는 다양한 표현 방식으로 같은 의미를 전달할 수 있습니다.

다음에 AI와 대화할 때, AI가 단순한 응답을 하더라도 그 뒤에는 복잡한 토큰과 확률 계산이 있다는 점을 기억해보세요!

<br>
<br>