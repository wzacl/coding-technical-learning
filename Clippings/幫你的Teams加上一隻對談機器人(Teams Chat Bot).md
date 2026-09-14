---
title: "幫你的Teams加上一隻對談機器人(Teams Chat Bot)"
source: "https://studyhost.blogspot.com/2018/07/teamsteams-chat-bot.html"
author:
  - "[[DD]]"
published: 2018-07-20
created: 2026-09-14
description: "前幾天我們介紹過了 如何申請免費 的Teams，但你知道嗎? 免費的Teams 其實也可以擁有對談機器人喔。只需要三分鐘就可以搞定… 上圖就是我們跟這個剛用三分鐘做好的機器人對談的畫面。 他現在只會Echo，啥都不會，但你幾乎不用寫任何程式碼就可以完成，這樣的對談機器人有何用途呢..."
tags:
  - "clippings"
---
前幾天我們介紹過了 [如何申請免費](http://studyhost.blogspot.com/2018/07/teams.html) 的Teams，但你知道嗎? 免費的Teams 其實也可以擁有對談機器人喔。只需要三分鐘就可以搞定…

![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_uUxB9F5FpUubm7nLk59bL_N8Ow1uKX51cZq7L77GqvPwhg8W7iKrKGYVoW9e3XBW4LZc0tzl_QztJVnVIVYMnVI6tNKWOTjjzNUDBw8ELt3h0ZIKBBF7bber7Vzae3WCrUJ_MpAEggrXlESlN1AWsW0Mj_DIk-6OrFKszCDkqzP84p90Fwsw=s0-d)

上圖就是我們跟這個剛用三分鐘做好的機器人對談的畫面。

他現在只會Echo，啥都不會，但你幾乎不用寫任何程式碼就可以完成，這樣的對談機器人有何用途呢?

用途大了，未來只需要稍加擴充，他就可以成為您在組織或企業內的小幫手，小從幫你查詢某人的分機、或是查詢你的今年度請假時數，大到自動幫你跑流程建立訂單或其他申請單，都是可以輕易實現的。

我們先來看如何建立一支這樣的bot。

## 使用bot service

要建立一支bot，你可以透過微軟Azure上的bot Service，建立時，目前有三種可以選擇：  
![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_s_oQIfe1154oLzmurToUzus4vwAqQ1oxX2czpMjuBljVvQfrSDR4dpafNBCdJoLdZO3MccIizPAc-fLeWSa7gmnJQ2gg6ILExBUyaQ2CKeC5AWw6mVU-xI1RMz_dPZzxddpBeoZMp-s4pSikQTnw2v3bV9nbbAyNQiRZZYzVFlE-O9WbBjEL0=s0-d)

請選擇最簡單的Web App bot，然後在出現的畫面中逐一填入相關資訊：

![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_v-H2NcfwmTUURzoX5bR7U6gZoEFSPHRUQ5t_BbdONAvg6j1X5oA5WX6fS3xTDxEIJ0i7EXTlFT_muHwVXcaxhMjUWeBzCDuWPl6qEoJHLsFhoQFrNh4enR3gaBLtisehfF0xqGjMH7Ip5T2t2cSc4_T2KY4U_8ZwAuvNwS92viM9w1AEaNYUQ=s0-d)

上圖1的機器人名稱當然就填寫你想要的名稱，而2,3,5預設會用同樣的名稱，我會建議你別改，且最好建立一個獨立的資源群組，這樣比較好管理。因為這個Web App Bot，會一口氣幫你建立Web App、Azure Storage、然後又可能有App Service Plan，如果你沒把所有東西放在一個獨立的資源群組裡，未來刪除的時候可能會漏東漏西。

上圖4程式碼的部分我們選擇C#，6的部分我們選擇自動建立。

按下『建立』鈕會跑一陣子，完成後，會出現類似底下這樣的畫面(如果沒有，請自行搜尋到該服務)：  
![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_vRqPMlzBJ1L1oelmXzlDQaTzrBQ3rvEXJbDcsypmJr5BgzKoa4H1Axpgrb50_Kf_I4ZnTKSUEGzuOSWpOoH_ei8w1Vo37v7NHWAZf-wZ1ZbBZYxmZuosK_j9tbTBKRW-wsgX6X8qDVaSFfZGIbEa3K_zYDi76mEHa_lm1mHWa8wBE9HFPrD_Y=s0-d)

接著，請點選『頻道』，我們要讓這個bot可以被加入Teams中，請在頻道中把teams家進去，完成後類似底下這樣：  
![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_sr5Gy4-aI2Kwxn9Gwv_d8AH1_BdfjaqL7ZZaeUmQzj0MpHA2wvuvlYCtmCqF1FZREgEEChSlwcEE5zK4SxCBifXabPuxIii8nN7giA7VxV56-HfpVnO4ZSMkpCRNIwppa34HM170GF9Pd95-5ff8hQj1xKeShO_24vJYCiMETxOD9f0N5nAS4=s0-d)

其實這樣該bot已經可以在Teams中使用了，但先別急，我們來看看這個bot目前是怎麼被設計的：

請在選單『組建』當中，找到開啟『線上代碼編輯器』：  
![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_viGJL8dWInY_s7VAGA6Q6ZIJ_9rs1aEjIMB8uExTiLyG4X8CZq2L3D0BbdfD16UIT9SrhD24IqZ-klpOlG5l0EDl5mSQiPLBbYsMsLER_EvQpLzUCqHzI8Kf53SOIifETE3QrP0vPJc6Icrz9NM-eDsQVlDJuWi2aZ9g1-rEgF2-IGcJRgrzA=s0-d)

點選後會出現底下這個Web畫面，基本上你可以在這個畫面中改你這隻Bot的程式碼，我們找到EchoDialog.cs：  
![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_v1vDtZ1jDJnqUc3Eb-MR_Ts_aLjoI9mdbe3sMiXfZS-rBMDTpfVTvGBBtnVcFJmiCW5HvFNA991Lz2C_dEihFG9sF5Ls2rX3_AgIRk6pN4mfCtIT0_kXcgbnW-x6q4sEPwqdM28pXKvw7AONxOsv0ew3MfZ0GwwwGGpibKzDFqHSxxW8-5nMY=s0-d)

然後把程式碼稍微改一下變成上面這樣:

```
await context.PostAsync($"{this.count++}: 親愛的，你說了 '{message.Text}' , 我有聽到......");
```

完成後，請點選左側選單中的Open Console，然後下指令 build.cmd ：

![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_uSHoSSC98yjckq4RZ7hVfZlVKekYZ_-nf1EYgmT8zo3g7xeerKT_UJXnoARlSHmmw_Wn-jgNVK0iqsjtMfiUNlIV4zTSjDpkU2314wZHtz2gQFvnQwXHIBX2wgVtqi20eV0iy8mTgkCy2kfSfVfT-QV692k-RouHb0TOnuQeHyQQQVyUg8zxY=s0-d)

完成後，你修改的程式碼就生效了!!!!!

這時候先別急，我們回到azure管理站台，找到MS App Id，這個ID很重要，是我們待會測試bot所需的關鍵資訊：

![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_vNZAFrKBAquQk4tWwMZ1WcQkTUwzlnD1lww9gXXZjY6mV7kpwUNCpFT8B7slPbd2F6AB7AWXhOX823PU1B-HSz8MURsP2cNe4_MgXfsUFIRmMcKQ0z1GC72yAQ-zPpL0ln7yMgseC0JKL-W3RygAUbtQNFcSW9PVGggln2lOhUpdLCiDZKCgg=s0-d)

找到之後，來到您的teams畫面，請在聊天選單中，點選下圖像是筆的那個圖示，在出現的『收件者』欄位中，輸入剛才你看到的MS App Id：

![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_uXiiKiEauJESUIY6f7pyDuE2jiRvOaUoG1U72w9Rvtu9kf8z2EAqOk2ckyH1ATtpGNs34uKQHwuHWWsTJ15MoUeoe_iC1pa6iOyWmvLlKRewvEZ4p2nraj638_tFDFuPQwW8Uy61zx51vp2ZAvD_33i4yCeo0xIN5QZ9tTIv9UGfrl7uCQZYg=s0-d)

你會發現這樣可以找到你設計好的bot，然後，你就可以跟他說話了：

![](https://lh3.googleusercontent.com/blogger_img_proxy/AEn0k_uUxB9F5FpUubm7nLk59bL_N8Ow1uKX51cZq7L77GqvPwhg8W7iKrKGYVoW9e3XBW4LZc0tzl_QztJVnVIVYMnVI6tNKWOTjjzNUDBw8ELt3h0ZIKBBF7bber7Vzae3WCrUJ_MpAEggrXlESlN1AWsW0Mj_DIk-6OrFKszCDkqzP84p90Fwsw=s0-d)

他當然也對答如流，只是言之無味， 後面就看你怎麼去改寫程式碼了。

三分鐘建立teams bot任務完成。