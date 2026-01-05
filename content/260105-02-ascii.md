+++
title = "YouTube ASCII 플레이어"
date = 2026-01-05
+++

<div style="text-align: center;">
  <input type="text" id="youtubeUrl" placeholder="YouTube URL 입력" style="width: 300px;">
  <button onclick="loadVideo()">로드</button>
  <br><br>
  <div id="player" style="display:none;"></div>
  <pre id="ascii" style="font-size: 5px; line-height: 5px; background: #000; color: #0f0;"></pre>
</div>

<script src="https://www.youtube.com/iframe_api"></script>
<script>
let player;
const ascii = document.getElementById('ascii');
const canvas = document.createElement('canvas');
const ctx = canvas.getContext('2d');
const chars = ' .\'`^",:;Il!i><~+_-?][}{1)(|\\/tfjrxnuvczXYUJCLQ0OZmwqpdbkhao*#MW&8%B@$';

const width = 120;
const height = 60;
canvas.width = width;
canvas.height = height;

function loadVideo() {
  const url = document.getElementById('youtubeUrl').value;
  const videoId = url.split('v=')[1]?.split('&')[0];
  
  if (player) player.destroy();
  
  player = new YT.Player('player', {
    height: '240',
    width: '320',
    videoId: videoId,
    events: { 'onReady': onPlayerReady }
  });
}

function onPlayerReady() {
  setInterval(captureFrame, 100);
}

function captureFrame() {
  // YouTube iframe에서는 직접 캡처가 불가능합니다
  // 대신 placeholder ASCII 애니메이션을 보여줍니다
  generateRandomAscii();
}

function generateRandomAscii() {
  let output = '';
  for (let y = 0; y < height; y++) {
    for (let x = 0; x < width; x++) {
      output += chars[Math.floor(Math.random() * chars.length)];
    }
    output += '\n';
  }
  ascii.textContent = output;
}
</script>

<p><em>참고: YouTube는 보안상 직접 프레임 캡처가 불가능해서 랜덤 ASCII로 시뮬레이션합니다.</em></p>