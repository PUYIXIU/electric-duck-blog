---
layout: wiki
wiki: BoyfriendtoDeath
title: 游戏选项
order: 0

---

{% folding 参考前需读 open:true color:warning %}
1. 攻略中出现的具体数值只是为了体现变化的{% mark 幅度 color:blue %}，是{% mark 目测值 color:warning %}，不是{% mark 精确值 color:error %}。
2. 攻略提供所有结局的{% mark 收集方式 color:warning %}，但不是{% mark 唯一方式 color:error %}。
3. 攻略不一定囊括所有可能出现的状况，待{% mark 后续补全❤️ color:red %}。
{% endfolding %}
   
{% folders %}
<!-- folder 1. So, what's your name? -->
- 'My name is xxx . Nice to meet you.' {% mark 无影响 color:light %}
- 'You first!' {% mark 无影响 color:light %}
- 'Oh ... uh my name is xxx.' {% mark ❤️+1 color:red%}

<!-- folder 2. You haven't seen your friends a while now? -->
- 'I haven't seen them in a while, but fuck it! I don't need them to have fun.'{% mark ❤️+1 color:red%}
- 'I see them once in a while but with finals, not as often,'  {% mark 无影响 color:light %}
- 'Not in a while. I've been trying to meet new people'{% mark ❤️+1 color:red%}

<!-- folder 3. 请选择你离开（被关小黑屋）的方式 -->
- {% folding I better head home. It's getting late. open:false color:blue %}
  - -Nod- {% mark 🩸 -5 color:orange%} {% mark 🧠-5 color:blue %}
  - Bite his hand {% mark ❤️+1 color:red%} {% mark 🩸 -25 color:orange%}
  - Struggle {% mark 🩸 -15 color:orange%} {% mark 🧠-5 color:blue %}
  {% endfolding %}
- {% folding I need to use the bathroom really quick! open:false color:green %}
  - Scream！ {% mark ❤️+2 color:red%} {% mark 🩸 -5 color:orange%} {% mark 🧠-5 color:blue %}
  - Move {% mark 🩸 -15 color:orange%} {% mark 🧠-5 color:blue %}
  {% endfolding %}  
- '...Hey, You wanna get out of here?' {% mark ❤️+1 color:red%} {% mark 🩸 -5 color:orange%} {% mark 🧠-5 color:blue %}

<!-- folder 4. I began to panic -->
- Yell for help {% mark ❤️+1 color:red%}
- Yell for Strade  {% mark 无影响 color:light %}
- Stay quiet {% mark 🧠-10 color:blue %}

<!-- folder 5. How ya feeling Snowfield? -->
- {% folding Stay quiet（❤️-1，🩸-10） open:false color:cyan %}
  - {% folding Stay quiet（🩸-15） open:false color:blue %}
      - Scream!  {% mark 无影响 color:light %}
      - {% folding 不采取行动（🩸-10，🧠-10） open:false color:purple %}
          - Stay quiet {% mark DE1 - Strade pulled your guts out！ color:error %}
          - I-it... hurts..  {% mark 无影响 color:light %}
        {% endfolding %}
    {% endfolding %}
  - It hurts...  {% mark 无影响 color:light %}
  {% endfolding %}
- My wrists hurt... {% mark ❤️+1 color:red%}
- What the fuck is going on !? Where am I !? {% mark ❤️+1 color:red%}

<!-- folder 6. I shifted on the floor uncomfortably. -->
- You  ABDUCTED me! Let me go!! {% mark ❤️+1 color:red%}
- I think there's been some kind of misunderstanding.  {% mark 无影响 color:light %}
- Please don't hurt me... {% mark ❤️+1 color:red%} {% mark 🧠-10 color:blue %}

<!-- folder 7. Strade问你开始之前要不要吃点喝点啥 -->
- I could use something to eat {% mark 🩸 +5 color:orange%} {% mark ❤️+1 color:red%}
- I'm thirsty {% mark ❤️+1 color:red%}  {% mark 🧠+5 color:blue %}
- Shake head  {% mark 无影响 color:light %}

<!-- folder 8. Strade小刀拉衣服 -->
- {% folding What? Wait! Stop!  open:false color:cyan %}
  - {% folding Ask him what he wants（❤️+1，🩸-10，🧠-25） open:false color:blue %}
    - Scream! {% mark ❤️+2 color:red%} {% mark 🧠-5 color:blue %} {% mark 🩸 -15 color:orange%}
    - 不采取行动 {% mark ❤️+1 color:red%} {% mark 🧠-10 color:blue %} {% mark 🩸 -15 color:orange%}
    {% endfolding %}
  - {% folding Say nothing（🩸-10，🧠-5） open:false color:purple %}
    - Scream! {% mark ❤️+2 color:red%} {% mark 🧠-5 color:blue %} {% mark 🩸 -15 color:orange%}
    - 不采取行动 {% mark ❤️+1 color:red%} {% mark 🧠-10 color:blue %} {% mark 🩸 -15 color:orange%}
    {% endfolding %}
  {% endfolding %}
- {% folding I'll do anything you want...（❤️+1） open:false color:red %}
    - {% folding Cut leg（❤️+1，🩸-5，🧠-10） open:false color:orange %}
      - {% folding More（❤️+1，🩸-10，🧠-15） open:false color:yellow %}
        - {% folding Do it（❤️+1，🩸-15，🧠-10） open:false color:green %}
          - Do it {% mark DE2-You did evetything he said color:error %}
          - 不采取行动 {% mark ❤️+1 color:red%} {% mark 🩸 -10 color:orange%}
          {% endfolding %}
        - 不采取行动 {% mark ❤️+1 color:red%} {% mark 🩸 -10 color:orange%}
        {% endfolding %}
      - 不采取行动 {% mark ❤️+1 color:red%} {% mark 🩸 -10 color:orange%}
      {% endfolding %}
    - {% folding Cut leg（❤️+1，🩸-5，🧠-10） open:false color:cyan %}
        - Scream! {% mark ❤️+2 color:red%} {% mark 🧠-5 color:blue %} {% mark 🩸 -40 color:orange%}
        - 不采取行动 {% mark ❤️+1 color:red%} {% mark 🧠-10 color:blue %} {% mark 🩸 -40 color:orange%}
      {% endfolding %}
  {% endfolding %}

<!-- folder 9. Would you like me to stitch those up for you? -->
- Ask him nicely {% mark ❤️+1 color:red%} {% mark 🧠-10 color:blue %} {% mark 🩸 +15 color:orange%}
- Nod reluctantly {% mark 🧠-10 color:blue %} {% mark 🩸 +15 color:orange%}
- {% folding Refuse（❤️+2，🩸-5，🧠-5） open:false color:cyan %}  
  - STOP！I'll take the stitches! {% mark ❤️+1 color:red%}  {% mark 🧠-10 color:blue %} {% mark 🩸 +15 color:orange%}
  - {% folding 不采取行动（触发特殊CG，❤️+1，🩸-5，🧠-5） open:false color:blue %}
    - Ignore the knife, bite down {% mark ❤️-2 color:red%} {% mark 🧠-10 color:blue %} {% mark 🩸 -20 color:orange%}
    - Try to avoid further injury {% mark ❤️+1 color:red%} {% mark 🧠-20 color:blue %} {% mark 🩸 +5 color:orange%}
    {% endfolding %}
  {% endfolding %}
      
<!-- folder 10. I can move around... what should I do? -->
- Get more rest {% mark 🧠+10 color:blue %} {% mark 🩸 +5 color:orange%}
- {% folding  Search fo food/water（🧠-20） open:false color:cyan %}
  - Drink a beer {% mark 🧠+10 color:blue %} {% mark 🩸 -5 color:orange%}
  - Eat the 'steak' {% mark 🩸 +5 color:orange%}
  - Drink and eat {% mark 🧠+10 color:blue %} {% mark 🩸 +5 color:orange%}
  - Don't touch anything {% mark 🩸 -5 color:orange%}
  - {% border color:red %}Tip：在这里如果🧠过低会触发DE13-You got the last laugh!{% endborder %}
  {% endfolding %}
- Search for a weapon {% mark 🔪 color:dark %} {% mark 🩸 -5 color:orange%}
- {% folding Try to escape（🧠-10） open:false color:blue %}
  - Go back to the pole {% mark 🩸 -5 color:orange%}
  - Try to sneak out {% mark DE3-Strade sawed your head in half color:error %} 
  {% endfolding %}

<!-- folder 11. Stradeの投食 -->
- Accept the food{% mark ❤️+2 color:red%} {% mark 🩸 +10 color:orange%}
- Refuse  {% mark 无影响 color:light %}

<!-- folder 12. Drill or hammer? -->
- {% folding hammer（❤️+1） open:false color:cyan%}
  {% folding 1. He grabbed my leg open:false color:blue%}
  - Kick him {% mark ❤️+2 color:red%} {% mark 🧠-50 color:blue %} {% mark 🩸 -25 color:orange%}
  - Stay still {% mark ❤️+2 color:red%} {% mark 🧠-45 color:blue %} {% mark 🩸 -25 color:orange%}
  {% endfolding %}
  {% folding 2. Too much? open:false color:purple%}
  - Say it hurts  {% mark DE11-You bled out ! color:error %} 
  - Stay Silent {% mark DE11-You bled out ! color:error %}
  - {% folding Spit at him（❤️+1，🩸被削到10，🧠被削到15） open:false color:red %}
    - Accept {% mark 🧠余5 color:blue %} {% mark 🩸余5 color:orange%}
    - Refuse {% mark DE11-You bled out ! color:error %} 
    {% endfolding %}
  {% endfolding %}
  {% endfolding %}
- {% folding drill（❤️+1，HEALYH-10，🧠-30） open:false color:orange%}
  {% folding 1. Scream open:false color:yellow%}
  - Scream！ {% mark ❤️+1 color:red%} {% mark 🧠-10 color:blue %} {% mark 🩸 -10 color:orange%}
  - 不采取行动 {% mark 🧠-10 color:blue %} {% mark 🩸 -20 color:orange%}
  {% endfolding %}
  {% folding 2. He smeared my own blood over my skin open:false color:green %}
  - {% folding Dont't touch me!（🩸-10） open:false color:cyan %}
    - {% folding Scream！ open:false color:orange%}
      - Scream {% mark ❤️+1 color:red%} {% mark 🩸 -10 color:orange%}
      - 不采取行动 {% mark 🧠-10 color:blue %} {% mark 🩸 -10 color:orange%}
      {% endfolding %}
    - Open your mouth
    - Do as he says {% mark 🧠-15 color:blue %} {% mark 🩸 -10 color:orange%}
    - Refuse {% mark 🧠-30 color:blue %} {% mark 🩸 -10 color:orange%}
    {% endfolding %}
  - {% folding say nothing（🧠-20，带小刀会触发特殊事件） open:false color:blue %}
    - Say Nothing {% mark DE4-Strade gave you knife back! color:error %} 
    - Apologize {% mark DE4-Strade gave you knife back! color:error %}
    - Admit planning to stab him {% mark DE4-Strade gave you knife back! color:error %} 
    {% endfolding %}
  - {% folding Moan（🧠较低时才会出现的选项，无影响） open:false color:purple %}
    {% border Tips：这里选择用小刀扎Strade会触发DE9-Strade stabbed you back! color:red %}{% endborder %}
    {% endfolding %}
  {% endfolding %}
  {% endfolding %}
- 不采取行动（Strade会随机选择hammer和drill） {% mark ❤️+1 color:red%} {% mark 🩸 -15 color:orange%}

<!-- folder 13. They were slow... and very light? -->
- Who's there?（Ren跑掉了）
- {% folding Stay quiet oepn:false color:green %}
  {% folding 1. Fox冒出来了 open:false color:blue%}
  - Who are you?
  - Please help Me！（Ren跑掉了）
  - Are those.. fox ears?
  {% endfolding %}
  {% folding 2. Fox说自己叫Ren open:false color:purple%}
  - {% folding Introduce yourself open:false color:yellow%}
    - You little asshole! He's going to KILL me!（Ren跑掉了）
    - I understand {% mark 🧠+10 color:blue %} {% mark 🩸 +15 color:orange%}
    {% endfolding %}
  - Please help me!（Ren跑掉了）
  - {% border  Tip：这里选择用小刀扎Ren会触发DE7-Strade axed your face! color:red%}{% endborder %}
  {% endfolding %}
  {% endfolding %}


<!-- folder 14. END -->
- 🩸较高，❤️是红心（存活结局）
- 🩸不够高 {% mark DE5-Strade lit you up! color:error %}
- ❤️不够高 {% mark DE6-Strade gave you another hole color:error %}

<!-- folder 其余死亡结局触发 -->
- 🧠为0 {% mark DE12-Strade made you a star! color:error %}
- ❤️爆表 {% mark DE10-Strade loved you too much color:error %}

{% endfolders %}
