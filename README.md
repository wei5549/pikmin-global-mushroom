<!DOCTYPE html>
<html lang="zh-Hant">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>全球 Level 3 找菇</title>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      background: #0b0d10;
      color: #f4f4f5;
      font-family: Arial, "Noto Sans TC", sans-serif;
    }

    .wrap {
      max-width: 1000px;
      margin: auto;
      padding: 28px 18px 60px;
    }

    h1 {
      font-size: 38px;
      margin: 15px 0 10px;
    }

    .sub {
      color: #9ca3af;
      line-height: 1.6;
      margin-bottom: 22px;
    }

    .panel {
      background: #15181e;
      border: 1px solid #2b3038;
      border-radius: 22px;
      padding: 18px;
      margin-bottom: 22px;
    }

    .controls {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 12px;
    }

    input,
    select,
    button {
      width: 100%;
      padding: 16px;
      border-radius: 14px;
      border: 1px solid #343a46;
      background: #11141a;
      color: white;
      font-size: 16px;
    }

    button {
      background: #1976d2;
      border: none;
      font-weight: bold;
      cursor: pointer;
    }

    button:disabled {
      opacity: 0.55;
      cursor: wait;
    }

    .check {
      display: flex;
      align-items: center;
      gap: 10px;
      margin-top: 16px;
      font-size: 16px;
    }

    .check input {
      width: 22px;
      height: 22px;
    }

    #status {
      margin-top: 18px;
      color: #aab1bd;
      line-height: 1.5;
    }

    .error {
      color: #ff7b7b !important;
    }

    .success {
      color: #78e08f !important;
    }

    .results {
      display: grid;
      gap: 14px;
    }

    .card {
      background: #15181e;
      border: 1px solid #2b3038;
      border-radius: 18px;
      padding: 18px;
    }

    .title {
      font-size: 21px;
      font-weight: bold;
      margin-bottom: 12px;
    }

    .row {
      margin: 7px 0;
      color: #d4d7dd;
      line-height: 1.5;
    }

    .label {
      color: #8e96a3;
    }

    .map {
      display: inline-block;
      margin-top: 12px;
      padding: 10px 14px;
      background: #1976d2;
      color: white;
      text-decoration: none;
      border-radius: 10px;
    }

    .count {
      margin: 0 0 14px;
      color: #aab1bd;
    }

    @media (max-width: 650px) {
      h1 {
        font-size: 31px;
      }

      .controls {
        grid-template-columns: 1fr;
      }
    }
  </style>
</head>

<body>

<div class="wrap">

  <h1>🍄 全球 Level 3 找菇</h1>

  <div class="sub">
    全球 Pikmin Bloom 蘑菇資料搜尋。<br>
    僅顯示 Level 3 蘑菇。
  </div>

  <div class="panel">

    <div class="controls">

      <input
        id="search"
        type="text"
        placeholder="搜尋國家、城市、地標..."
      >

      <select id="type">
        <option value="">全部菇種類</option>
        <option value="2">🔴 大紅菇</option>
        <option value="5">🔵 大藍菇</option>
        <option value="6">🟡 大黃菇</option>
        <option value="7">⚪ 大白菇</option>
        <option value="8">🟣 大紫菇</option>
        <option value="9">🩷 大粉菇</option>
        <option value="11">🔥 大火菇</option>
        <option value="12">💧 大水菇</option>
        <option value="13">💎 大水晶菇</option>
        <option value="17">⚡ 大電菇</option>
        <option value="18">☠️ 大毒菇</option>
        <option value="26">🧊 大冰藍菇</option>
      </select>

      <select id="sort">
        <option value="updated">最近更新</option>
        <option value="ending">最快結束</option>
        <option value="participants">參加人數最少</option>
      </select>

      <button id="reload">
        重新抓取資料
      </button>

    </div>

    <label class="check">
      <input id="joinable" type="checkbox" checked>
      只顯示還能加入
    </label>

    <div id="status">
      準備讀取資料...
    </div>

  </div>

  <div id="count" class="count"></div>
  <div id="results" class="results"></div>

</div>


<script>

const API =
  "https://late-glade-6923.awho86958.workers.dev";

let allData = [];

const searchEl = document.getElementById("search");
const typeEl = document.getElementById("type");
const sortEl = document.getElementById("sort");
const joinableEl = document.getElementById("joinable");

const reloadBtn = document.getElementById("reload");
const statusEl = document.getElementById("status");
const resultsEl = document.getElementById("results");
const countEl = document.getElementById("count");


const TYPE_NAMES = {
  2: "🔴 大紅菇",
  5: "🔵 大藍菇",
  6: "🟡 大黃菇",
  7: "⚪ 大白菇",
  8: "🟣 大紫菇",
  9: "🩷 大粉菇",
  11: "🔥 大火菇",
  12: "💧 大水菇",
  13: "💎 大水晶菇",
  17: "⚡ 大電菇",
  18: "☠️ 大毒菇",
  26: "🧊 大冰藍菇"
};


function esc(value) {
  return String(value ?? "")
    .replaceAll("&", "&amp;")
    .replaceAll("<", "&lt;")
    .replaceAll(">", "&gt;")
    .replaceAll('"', "&quot;")
    .replaceAll("'", "&#039;");
}


function first(obj, keys, fallback = "") {

  for (const key of keys) {

    if (
      obj &&
      obj[key] !== undefined &&
      obj[key] !== null
    ) {
      return obj[key];
    }

  }

  return fallback;
}


function normalize(raw) {

  const level = Number(
    first(raw, [
      "level",
      "mushroom_level",
      "mushroomLevel"
    ], 0)
  );

  const type = Number(
    first(raw, [
      "type",
      "mushroom_type",
      "mushroomType"
    ], 0)
  );

  const lat = Number(
    first(raw, [
      "lat",
      "latitude"
    ], 0)
  );

  const lng = Number(
    first(raw, [
      "lng",
      "lon",
      "longitude"
    ], 0)
  );

  const participants = Number(
    first(raw, [
      "participants",
      "participant_count",
      "participantCount",
      "challenger_count"
    ], 0)
  );

  const maxParticipants = Number(
    first(raw, [
      "max_participants",
      "maxParticipants",
      "participant_limit",
      "challenger_capacity"
    ], 0)
  );

  const country = first(raw, [
    "country",
    "location_country"
  ], "");

  const city = first(raw, [
    "city",
    "location_city"
  ], "");

  const name = first(raw, [
    "name",
    "mushroom_name",
    "spot_name",
    "location_name"
  ], "");

  const updated = Number(
    first(raw, [
      "updated_at",
      "last_observed_at",
      "last_seen",
      "discovered_at"
    ], 0)
  );

  const finish = Number(
    first(raw, [
      "finish_ms",
      "finish_at",
      "end_time",
      "endTime"
    ], 0)
  );

  return {
    raw,
    level,
    type,
    lat,
    lng,
    participants,
    maxParticipants,
    country,
    city,
    name,
    updated,
    finish
  };
}


function getArray
