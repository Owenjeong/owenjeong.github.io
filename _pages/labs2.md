---
layout: single
permalink: /labs/lab2
title: "Revenue by Rails_BI"
---

<style>
.powerbi-container {
  width: 100%;
  height: 100vh; /* 전체 화면 높이 */
  margin: 0;
  padding: 0;
}

.powerbi-container iframe {
  width: 100%;
  height: 100%;
  border: none;
}

/* 네비게이션 바 높이 고려해서 조정 */
.powerbi-fullscreen {
  width: 100vw;
  height: calc(100vh - 80px); /* 헤더 높이만큼 빼기 */
  margin-left: calc(-50vw + 50%);
}
</style>

<div class="powerbi-fullscreen">
  <iframe width="100%" 
          height="100%" 
          src="https://app.powerbi.com/reportEmbed?reportId=7e2899b9-0538-4690-b968-af46ff268353&autoAuth=true&ctid=44467e6f-462c-4ea2-823f-7800de5434e3" 
          frameborder="0" 
          allowFullScreen="true">
  </iframe>
</div>