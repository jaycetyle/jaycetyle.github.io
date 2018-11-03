---
layout: post
title: Synology Data Scrubbing 介紹與分析
description: "最近有人在詢問 Synology 的 Data Scrubbing 相關的問題，剛好小弟算是小有研究，因此來介紹一下何謂 Data Scrubbing（資料洗滌），以及 Synology 的 Data Scrubbing 可以為我們做些什麼？"
modified: 2018-11-03
tags: [小法師]
categories: [NAS 與資料備份]
image:
    feature: 
    credit: 
    creditlink: 
---

　　最近有人在詢問 Synology 的 Data Scrubbing 相關的問題，剛好小弟算是小有研究，因此來介紹一下何謂 Data Scrubbing（資料洗滌），以及 Synology 的 Data Scrubbing 可以為我們做些什麼？

<!--more-->　

## 何謂 RAID？

　　講到 Data Scrubbing 勢必得提一下什麼是 RAID。RAID 在 wiki 上的中文翻譯是[容錯式磁碟陣列](https://zh.wikipedia.org/wiki/RAID)，簡單來說，RAID 是將多顆硬碟組合起來形成一個較大的空間，從上層的檔案系統角度來看，他就像一顆硬碟一樣。而 RAID 根據硬碟資料組合的方法不同，還可以分為 RAID0、RAID1、RAID5 以及 RAID6 等模式，一般家用 RAID 最常用的就是 RAID1 和 RAID5 這兩種，以下簡單介紹這兩種類型。

#### RAID1

　　RAID1 是最容易理解的 RAID 模式，他需要至少 2 顆硬碟參與，所有參與的硬碟彼此互相做鏡像，因此陣列中如果有硬碟損壞，只要至少還保有一顆硬碟，資料就仍然可以保留，但相對的空間使用率會較低，因為無論參與的硬碟有幾顆，組合而成磁碟陣列大小就是一顆硬碟的大小。

<figure class="third center">
<img src="/images/2018/11/synology-scrubbing-01.png" alt="">
</figure>

　　RAID 1 在寫入時，資料同時會對所有硬碟進行寫入；讀取時，由於所有硬碟有著相同資料，所以可從任一顆硬碟內讀取，一般系統會進行一定程度的負載平衡 (load balance)，當其中一顆較忙碌時會切換到另外一顆進行讀取，以提高效能。RAID1 也是 2bay (2硬碟插槽) NAS 的常見選擇。

#### RAID5

　　RAID5 稍微複雜一些，需要至少 3 顆以上的硬碟參與，他允許陣列中的任意一顆硬碟或損毀，而故障硬碟的原始內容可以由另外兩顆硬碟計算恢復，恢復的原理是使用奇偶校驗資訊。

　　首先 RAID5 會對硬碟位置進行位址劃分，在我們要寫入一塊連續資料到陣列時，他會依照 A1, A2, A3, B1, B2, B3 ... 的順序寫入到硬碟中時；而讀取時，同樣依照 A1, A2, A3, B1, B2, B3 ... 讀取，如下圖：

<figure class="mid center">
<img src="/images/2018/11/synology-scrubbing-02.png" alt="">
</figure>

　　那圖中的 Ap, Bp, Cp 呢？這些就是所謂的奇偶校驗區塊 (Parity)。例如RAID5 在寫入 A1 ~ A3 的同時，他還會透過一種特殊的數學式 XOR (異或) 計算出 Ap 的內容並寫入到對應位置中，如下：

    Ap = A1 (XOR) A2 (XOR) Ap　　(式一)

　　當 A1 ~ A3 的其中一顆硬碟故障時，RAID5 可以再利用同樣的數學式 XOR，結合 Ap 及另外的兩顆硬碟內容將遺失的還原出來。例如 A2 所屬硬碟故障了，A2 的內容可以透過以下算式計算而得：

    A2 = A1 (XOR) A3 (XOR) Ap　　(式二)

　　我們把這種計算還原出來的內容稱為冗餘備份 (Redundant Copy)，RAID5 便是以此方法達到資料保護的目的，很神奇吧！RAID5 同時也是 4bay NAS 的常見選擇，相較於 RAID1 和完全沒有保護這兩種極端情況，RAID5 在空間利用率和安全性中有著不錯的平衡。

　

## 資料一致性 (Data Consistency) 與 RAID Scrubbing

　　我們知道 RAID5 的特性了，接下來我們就可以說明 RAID 的資料一致性。首先我們知道 RAID5 每顆硬碟的內容有著 式一 的數學關係式，如果關係式滿足的話，我們就說資料在 RAID 中是一致的，因為當有硬碟故障時，該內容才可以用 式二 計算出冗餘備份還原回來；如果沒有滿足的話，我們就稱之為不一致，此時透過 XOR 的關係式求回來的內容就會是錯的而還原失敗。

　　資料還原失敗是很嚴重的一件事！因此保持 RAID 資料一致性就很重要，RAID Scrubbing 會掃描磁碟陣列中的所有內容，確保所有的奇偶校驗區塊都滿足 式一，如果不滿足的話就重新算一次把他修正到一致，讓我們保有一致又健康的磁碟陣列，所以定期排程做 Scrubbing 是很重要的。

　

## 檔案系統的 Data Scrubbing

　　定期排程 Scrubbing，資料就可以受到 RAID 妥善的保護，這樣就可以高枕無憂了嗎？很可惜這個答案是**否定**的，因為資料並不是寫入硬碟之後就永遠會保持正確，硬碟有時候會遭遇一些沒有預兆的資料錯誤 (Silent Data Error)，有可能是物理或電磁特性種種原因，導致原本儲存的檔案內容突然就變了，這種情況就算做過 RAID Scrubbing 也沒用，因為 RAID 無法辨別哪個區塊的資料是錯的，他只能讓資料保持一致，資料一致不等於資料正確，如果其中有人錯了，大家也會錯得一致。

　　那該怎麼辦呢？此時就要用到 **btrfs** 檔案系統的 Data Scrubbing 功能，眼尖的使用者可能會注意到儲存空間管理員的 Data Scrubbing 頁面內寫了兩種 Scrubbing，其中 RAID Scrubbing 指的是先前說明有關 RAID 一致性的 Scrubbing，而 Data Scrubbing 指的就是 btrfs 檔案系統的 Scrubbing。

<figure class="center">
<img src="/images/2018/11/synology-scrubbing-03.png" alt="">
</figure>

　　btrfs 檔案系統的 Data Scrubbing 是利用檔案的總和檢查碼 (checksum) 做檔案檢查。如果在建立共用資料夾的時候有勾選"資料總和檢查碼以進行進階資料保護"的功能的話，檔案系統會為所有寫入的檔案都計算總和檢查碼，並存在檔案系統的中繼資料 (metadata) 中，且中繼資料存有兩份。不過多開啟這個功能是有輕微效能代價的，如果是存放 VM 或是資料庫的共享資料夾，建議是不要開啟這個功能，但存放的是一般文件或是照片的話，對效能影響非常小，所以家用的使用者都可以放心，也務必要打開他！

<figure class="large center">
<img src="/images/2018/11/synology-scrubbing-04.png" alt="">
</figure>

　　在進行 Data Scrubbing 時，檔案系統會為重新計算檔案的檢查碼，如果發現檢查碼跟原本計算的不同，那有兩種可能 1. 檔案內容錯了 2. 中繼資料存的原始總和檢查碼錯了。由於中繼資料有兩份，因此檔案系統會再比對第二份的檢查碼，如果兩份檢查碼結果一樣，就可以推斷應該是檔案錯了，如果檢查碼不同，則可以推斷應該是其中一份中繼資料出錯了。

　　接下來 btrfs 會開始嘗試修復出錯的檔案，而修復的方式就是從底下的 RAID 取得備份，像 RAID5 就是用 Parity 計算冗餘備份，RAID1 則是直接取得鏡像資料，並計算備份的總和檢查碼。如果總和檢查碼正確的話，就可以用備份的資料來修復原始資料。下圖為修復的流程範例：

<figure class="center">
<img src="/images/2018/11/synology-scrubbing-05.png" alt="">
</figure>

　　Synology 的 Data Scrubbing 實際上便是依序執行了檔案系統 Scrubbing 及 RAID Scrubbing。檔案系統的 Scrubbing 先確保了資料的正確性，而後續的 RAID Scrubbing 則會確保資料的一致性，這樣就可以同時有既正確又一致的健康系統！

　

## 常見 Q/A
**Q: 我的 RAID 類型是 RAID1，為什麼顯示不支援 RAID Scrubbing 呢？**

A: 因為 RAID1 不需要、也無法做 RAID Scrubbing 哦！RAID1 平常在讀取的時候是隨機從兩顆的其中一顆讀取，當 RAID1 Scrubbing 發現不一致時，實際上也無法決定要如何讓他變一致。而 RAID5 在讀取時都是讀資料區塊 (A1, A2, A3, B1...)，而非校驗區塊 (Ap, Bp...)，原則上 RAID5 在 Scrubbing 是選擇 *相信資料區塊*，因此在做 RAID5 Scrubbing 時，實際上是在檢查並更新校驗區塊。且之前做的檔案系統 Scrubbing 已經可以確保資料區塊是正確的，因此 RAID5 也可以相信資料區塊是正確的。

**Q: 我的檔案系統是 ext4，有支援檔案系統層級的 Data Scrubbing 嗎？**

A: 沒有哦！ext4 的檔案系統沒有儲存檔案總和校驗碼的功能，也不支援 Data Scrubbing，趕快找個機會換到 btrfs 吧！

**Q: 我是他牌的使用者，請問有支援 Data Scrubbing 嗎？**

A: 我沒用過其他廠牌哦！歡迎貢獻一台給我，我*不一定*會想研究看看！

　

---

本文感謝 [BingJing Chang](https://www.linkedin.com/in/bingjing-chang/) 的技術協助！
