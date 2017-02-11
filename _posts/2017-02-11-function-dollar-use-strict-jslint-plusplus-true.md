---
description: "(function ($) {\r\n  \"use strict\";\r\n  /*jslint plusplus: true */\r\n  /*jslint browser: true*/\r\n  /*jslint newcap: true */\r\n  /*global $, jQuery, alert, Audio, case, wptpa_data*/\r\n  /*jslint evil: true */\r\n  /*global console */\r\n  /*global Snap */\r\n  /*global mina */\r\n  $.fn.tPlayer = function (options) {\r\n    // Дефолтные настройки\r\n    var defaults = {\r\n      playlist: '',\r\n      autoplay: false,\r\n      showPlaylist: true,\r\n      shuffle: true,\r\n      share: true,\r\n      twitterText: '',\r\n      compactMode: false,\r\n      defaultVolume: 0.75,\r\n      promo: '',\r\n      promoMode: false,\r\n      promoTime: 60000,\r\n      songlimit: 0,\r\n      playerBG: \"#fff\",\r\n      playerTextCLR: \"#555\",\r\n      buttonCLR: \"#555\",\r\n      buttonActiveCLR: \"#3EC3D5\",\r\n      seekBarCLR: \"#555\",\r\n      progressBarCLR: \"#3EC3D5\",\r\n      timeCLR: \"#fff\",\r\n      playlistBG: \"#3EC3D5\",\r\n      playlistTextCLR: \"#fff\",\r\n      playlistCurBG: \"#42CFE2\",\r\n      playlistTextCurCLR: \"#fff\"\r\n    },\r\n      settings = $.extend(defaults, options), // обьеденяем деволтные настройки с пользовательскими настройками\r\n      $tPlayer = $(this), // Обьявляем главный контейнер\r\n      playlist = JSON.parse(JSON.stringify(settings.playlist)),// Превращаем обьект в JSON, а джосн строку в массив\r\n      // обьявляем переменные для контейнеров\r\n      $player, $cover_viewer, $cover_wrapper, $cover, $heading, $marquee, $playback, $audio, $seek, $progress, $current,\r\n      $duration, $previous, $repeat, $next, $shuffle, $share, $playlist_toggle, $mute, $volume_range,\r\n      $volume, $volumeValue, $playlist, $playlistItem,  $wptpa_playlist_item, $wptpa_playlist_scroll, $wptpa_playlist_scroll_handle, $wptpa_dwn, $fb, $twitter, $gp, $tmblr, $modal, $overlay, $info_wrapper,\r\n      $info_heading, $info_container, $promo, promoInterval, coverside, open_nw, playbackIcn, seekHandleIcn,\r\n      nextIcn, repeatIcn, prevIcn, shuffleIcnP, shuffleIcnAT, shuffleIcnAB, shareIcnP, shareIcnCT, shareIcnCM, shareIcnCB,\r\n      playlist_toggleIcn, muteIcn, mutedIcn, muteIcnL, muteIcnM, muteIcnR, playIcon, $wptpa_cover_loader, $info_area, all_count, all_dwn,\r\n      \r\n      songlimit = settings.songlimit, // получаем лимит плейлиста\r\n      \r\n      // обьявляем переменные и флаги\r\n      currentSong, // Текущий трек\r\n      currentSlide, // Текущий слайд\r\n      countSongs, // кол-во треков\r\n      playlistOrder = '', //скопировать порядок песен для возврата с шафла\r\n      coverOrder = '', //скопировать порядок обложек для возврата с шафла\r\n      repeatMode = false,\r\n      shuffleMode = false,\r\n      seeksliding = false,\r\n      mutedAnim = false,\r\n      adsMode = false,\r\n      volume = settings.defaultVolume, // громкость\r\n      showPlaylist = settings.showPlaylist, // показывать ли плейлист\r\n      $code = '',\r\n      i = 0, title, tips,\r\n      countCover = 0,\r\n      style = '',\r\n      id = $tPlayer.attr(\"id\"); // кол-во каверов для проверки их загружености\r\n    \r\n    style = '';\r\n    // добавляем скрол если установлен\r\n    if (settings.songlimit  0) {\r\n      style += '#' + id + ' .wptpa_playlist_wrapper { max-height:' + (settings.songlimit * 40) + 'px; }';\r\n    }\r\n\r\n    style += \"#\" + id + \" .wptpa_social_share,\";\r\n    style += \"#\" + id + \" .wptpa_player {\";\r\n    style += \"background: \" + settings.playerBG + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_cover_viewer {\";\r\n    style += \"background: \" + settings.seekBarCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_circular_path{\";\r\n    style += \"stroke: \" + settings.progressBarCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_cover_viewer:before {\";\r\n    style += \"background: \" + settings.playerBG + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_cover_viewer .wptpa_elastic_side path {\";\r\n    style += \"fill: \" + settings.playerBG + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_controls {\";\r\n    style += \"background: \" + settings.playerBG + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_heading {\";\r\n    style += \"color: \" + settings.playerTextCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_marquee.on {\";\r\n    style += \"border-left-color: \" + settings.playerTextCLR + \";\";\r\n    style += \"border-right-color: \" + settings.playerTextCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_heading .wptpa_open_nw,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_playback,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_prev,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_repeat,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_next,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_shuffle,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_share,\";\r\n    style += \"#\" + id + \" .wptpa_console .wptpa_close_console,\";\r\n    style += \"#\" + id + \" .wptpa_additional_controls .wptpa_one_item_optional,\";\r\n    style += \"#\" + id + \" .wptpa_additional_controls .wptpa_playlist_toggle,\";\r\n    style += \"#\" + id + \" .wptpa_additional_controls .wptpa_mute,\";\r\n    style += \"#\" + id + \" .wptpa_additional_controls .wptpa_one_item_optional{\";\r\n    style += \"fill: \" + settings.buttonCLR + \";\";\r\n    style += \"stroke: \" + settings.buttonCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_heading .wptpa_open_nw:hover,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_playback:hover,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_playback:hover,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_prev:hover,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_repeat:hover,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_next:hover,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_shuffle:hover,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_share:hover,\";\r\n    style += \"#\" + id + \" .wptpa_console .wptpa_close_console:hover,\";\r\n    style += \"#\" + id + \" .wptpa_additional_controls .wptpa_one_item_optional:hover,\";\r\n    style += \"#\" + id + \" .wptpa_additional_controls .wptpa_playlist_toggle:hover,\";\r\n    style += \"#\" + id + \" .wptpa_additional_controls .wptpa_mute:hover,\";\r\n    style += \"#\" + id + \" .wptpa_additional_controls .wptpa_one_item_optional:hover,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_repeat.active,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_shuffle.active,\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_share.active,\";\r\n    style += \"#\" + id + \" .wptpa_additional_controls .wptpa_playlist_toggle.active{\";\r\n    style += \"fill: \" + settings.buttonActiveCLR + \";\";\r\n    style += \"stroke: \" + settings.buttonActiveCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_seek {\";\r\n    style += \"background: \" + settings.seekBarCLR + \" !important;\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \".wptpa_loading .wptpa_seek:before,\";\r\n    style += \"#\" + id + \".wptpa_loading .wptpa_seek:after{\";\r\n    style += \"background: \" + settings.timeCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_progress {\";\r\n    style += \"background: \" + settings.progressBarCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_progress .wptpa_handle{\";\r\n    style += \"fill: \" + settings.progressBarCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_main_controls .wptpa_handle .handlebg{\";\r\n    style += \"fill: \" + settings.seekBarCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_current,\";\r\n    style += \"#\" + id + \" .wptpa_duration,\";\r\n    style += \"#\" + id + \" .wptpa_sng_info{\";\r\n    style += \"color: \" + settings.timeCLR + \";\";\r\n    style += \"}\";\r\n\r\n    style += \"#\" + id + \" .wptpa_volume_range:before {\";\r\n    style += \"background: \" + settings.seekBarCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_volume {\";\r\n    style += \"background: \" + settings.progressBarCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_volume_value {\";\r\n    style += \"background: \" + settings.progressBarCLR + \";\";\r\n    style += \"color: \" + settings.timeCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_volume_value:before {\";\r\n    style += \"border-color: \" + settings.progressBarCLR + \" transparent transparent transparent;\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_social_share svg{\";\r\n    style += \"fill: \" + settings.buttonCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_social_share svg:hover{\";\r\n    style += \"fill: \" + settings.buttonActiveCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_playlist_item,\";\r\n    style += \"#\" + id + \" .wptpa_playlist_scroll_handle {\";\r\n    style += \"background: \" + settings.playlistBG + \";\";\r\n    style += \"color: \" + settings.playlistTextCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_playlist_item:hover,\";\r\n    style += \"#\" + id + \" .wptpa_playlist_item.active{\";\r\n    style += \"background: \" + settings.playlistCurBG + \" !important;\";\r\n    style += \"color: \" + settings.playlistTextCLR + \" !important;\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_playlist_item .wptpa_marker {\";\r\n    style += \"stroke: \" + settings.playlistTextCLR + \";\";\r\n    style += \"}\";\r\n    style += \"#\" + id + \" .wptpa_stat_icon_sng,\";\r\n    style += \"#\" + id + \" .wptpa_buy svg,\";\r\n    style += \"#\" + id + \" .wptpa_download svg {\";\r\n    style += \"fill: \" + settings.playlistTextCLR + \";\";\r\n    style += \"stroke: \" + settings.playlistTextCLR + \";\";\r\n    style += \"}\";\r\n\r\n    style += '';\r\n    $(\"head\").append(style);\r\n      \r\n    // Формирируем код плеера\r\n    $code = ''; // конейнер для аудио, Сразу вставляем первый трек\r\n    // Если, включен компакт режим присваеваем основному контейнеру его класс\r\n    if (settings.compactMode) {\r\n      $tPlayer.addClass('wptpa_compact');\r\n    }\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    // в цикле вставляем обложки в слайдер\r\n    for (i = 0; i ';\r\n    }\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '';\r\n    $code += '00:0000:00';\r\n    \r\n    // Если треков больше одного - выводим кнопки prev, repeat, next\r\n    if (playlist.length  1) {\r\n      $code += '';\r\n      $code += '';\r\n      $code += '';\r\n    }\r\n    // Если треков больше одного и опция shuffle разрешена - выводим ее кнопку\r\n    if (playlist.length  1 && settings.shuffle) {\r\n      $code += '';\r\n    }\r\n    // Если треков больше одного и опция share разрешена - выводим ее кнопку\r\n    if (settings.share) {\r\n      $code += '';\r\n    }\r\n    $code += '';\r\n    $code += '';\r\n    // Если треков больше одного выводим кнопку toggle playlist, если трек один, в зависимости от значения опционального типа выводим - либо кнопку скачать, либо купить, либо заглушку для верстки\r\n    if (playlist.length  1) {\r\n      $code += '';\r\n    } else {\r\n      \r\n      if (!playlist[0].dwn) {\r\n        playlist[0].dwn = \"\";\r\n      }\r\n      if (!playlist[0].buy) {\r\n"
author: []
datePublished: '2017-02-11T17:43:04.977Z'
dateModified: '2017-02-11T17:43:04.383Z'
datePublishedOriginal: '2017-02-11T17:43:04.977Z'
title: ''
publisher: {}
via: {}
inFeed: true
starred: false
sourcePath: _posts/2017-02-11-function-dollar-use-strict-jslint-plusplus-true.md
_context: 'http://schema.org'
_type: Article

---
(function ($) {
"use strict";
/\*jslint plusplus: true \*/
/\*jslint browser: true\*/
/\*jslint newcap: true \*/
/\*global $, jQuery, alert, Audio, case, wptpa\_data\*/
/\*jslint evil: true \*/
/\*global console \*/
/\*global Snap \*/
/\*global mina \*/
$.fn.tPlayer = function (options) {
// Дефолтные настройки
var defaults = {
playlist: '',
autoplay: false,
showPlaylist: true,
shuffle: true,
share: true,
twitterText: '',
compactMode: false,
defaultVolume: 0.75,
promo: '',
promoMode: false,
promoTime: 60000,
songlimit: 0,
playerBG: "\#fff",
playerTextCLR: "\#555",
buttonCLR: "\#555",
buttonActiveCLR: "\#3EC3D5",
seekBarCLR: "\#555",
progressBarCLR: "\#3EC3D5",
timeCLR: "\#fff",
playlistBG: "\#3EC3D5",
playlistTextCLR: "\#fff",
playlistCurBG: "\#42CFE2",
playlistTextCurCLR: "\#fff"
},
settings = $.extend(defaults, options), // обьеденяем деволтные настройки с пользовательскими настройками
$tPlayer = $(this), // Обьявляем главный контейнер
playlist = JSON.parse(JSON.stringify(settings.playlist)),// Превращаем обьект в JSON, а джосн строку в массив
// обьявляем переменные для контейнеров
$player, $cover\_viewer, $cover\_wrapper, $cover, $heading, $marquee, $playback, $audio, $seek, $progress, $current,
$duration, $previous, $repeat, $next, $shuffle, $share, $playlist\_toggle, $mute, $volume\_range,
$volume, $volumeValue, $playlist, $playlistItem, $wptpa\_playlist\_item, $wptpa\_playlist\_scroll, $wptpa\_playlist\_scroll\_handle, $wptpa\_dwn, $fb, $twitter, $gp, $tmblr, $modal, $overlay, $info\_wrapper,
$info\_heading, $info\_container, $promo, promoInterval, coverside, open\_nw, playbackIcn, seekHandleIcn,
nextIcn, repeatIcn, prevIcn, shuffleIcnP, shuffleIcnAT, shuffleIcnAB, shareIcnP, shareIcnCT, shareIcnCM, shareIcnCB,
playlist\_toggleIcn, muteIcn, mutedIcn, muteIcnL, muteIcnM, muteIcnR, playIcon, $wptpa\_cover\_loader, $info\_area, all\_count, all\_dwn,

songlimit = settings.songlimit, // получаем лимит плейлиста

// обьявляем переменные и флаги
currentSong, // Текущий трек
currentSlide, // Текущий слайд
countSongs, // кол-во треков
playlistOrder = '', //скопировать порядок песен для возврата с шафла
coverOrder = '', //скопировать порядок обложек для возврата с шафла
repeatMode = false,
shuffleMode = false,
seeksliding = false,
mutedAnim = false,
adsMode = false,
volume = settings.defaultVolume, // громкость
showPlaylist = settings.showPlaylist, // показывать ли плейлист
$code = '',
i = 0, title, tips,
countCover = 0,
style = '',
id = $tPlayer.attr("id"); // кол-во каверов для проверки их загружености

style = '';
// добавляем скрол если установлен
if (settings.songlimit \> 0) {
style += '\#' + id + ' .wptpa\_playlist\_wrapper { max-height:' + (settings.songlimit \* 40) + 'px; }';
}

style += "\#" + id + " .wptpa\_social\_share,";
style += "\#" + id + " .wptpa\_player {";
style += "background: " + settings.playerBG + ";";
style += "}";
style += "\#" + id + " .wptpa\_cover\_viewer {";
style += "background: " + settings.seekBarCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_circular\_path{";
style += "stroke: " + settings.progressBarCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_cover\_viewer:before {";
style += "background: " + settings.playerBG + ";";
style += "}";
style += "\#" + id + " .wptpa\_cover\_viewer .wptpa\_elastic\_side path {";
style += "fill: " + settings.playerBG + ";";
style += "}";
style += "\#" + id + " .wptpa\_controls {";
style += "background: " + settings.playerBG + ";";
style += "}";
style += "\#" + id + " .wptpa\_heading {";
style += "color: " + settings.playerTextCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_marquee.on {";
style += "border-left-color: " + settings.playerTextCLR + ";";
style += "border-right-color: " + settings.playerTextCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_heading .wptpa\_open\_nw,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_playback,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_prev,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_repeat,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_next,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_shuffle,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_share,";
style += "\#" + id + " .wptpa\_console .wptpa\_close\_console,";
style += "\#" + id + " .wptpa\_additional\_controls .wptpa\_one\_item\_optional,";
style += "\#" + id + " .wptpa\_additional\_controls .wptpa\_playlist\_toggle,";
style += "\#" + id + " .wptpa\_additional\_controls .wptpa\_mute,";
style += "\#" + id + " .wptpa\_additional\_controls .wptpa\_one\_item\_optional{";
style += "fill: " + settings.buttonCLR + ";";
style += "stroke: " + settings.buttonCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_heading .wptpa\_open\_nw:hover,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_playback:hover,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_playback:hover,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_prev:hover,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_repeat:hover,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_next:hover,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_shuffle:hover,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_share:hover,";
style += "\#" + id + " .wptpa\_console .wptpa\_close\_console:hover,";
style += "\#" + id + " .wptpa\_additional\_controls .wptpa\_one\_item\_optional:hover,";
style += "\#" + id + " .wptpa\_additional\_controls .wptpa\_playlist\_toggle:hover,";
style += "\#" + id + " .wptpa\_additional\_controls .wptpa\_mute:hover,";
style += "\#" + id + " .wptpa\_additional\_controls .wptpa\_one\_item\_optional:hover,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_repeat.active,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_shuffle.active,";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_share.active,";
style += "\#" + id + " .wptpa\_additional\_controls .wptpa\_playlist\_toggle.active{";
style += "fill: " + settings.buttonActiveCLR + ";";
style += "stroke: " + settings.buttonActiveCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_seek {";
style += "background: " + settings.seekBarCLR + " !important;";
style += "}";
style += "\#" + id + ".wptpa\_loading .wptpa\_seek:before,";
style += "\#" + id + ".wptpa\_loading .wptpa\_seek:after{";
style += "background: " + settings.timeCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_progress {";
style += "background: " + settings.progressBarCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_progress .wptpa\_handle{";
style += "fill: " + settings.progressBarCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_main\_controls .wptpa\_handle .handlebg{";
style += "fill: " + settings.seekBarCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_current,";
style += "\#" + id + " .wptpa\_duration,";
style += "\#" + id + " .wptpa\_sng\_info{";
style += "color: " + settings.timeCLR + ";";
style += "}";

style += "\#" + id + " .wptpa\_volume\_range:before {";
style += "background: " + settings.seekBarCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_volume {";
style += "background: " + settings.progressBarCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_volume\_value {";
style += "background: " + settings.progressBarCLR + ";";
style += "color: " + settings.timeCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_volume\_value:before {";
style += "border-color: " + settings.progressBarCLR + " transparent transparent transparent;";
style += "}";
style += "\#" + id + " .wptpa\_social\_share svg{";
style += "fill: " + settings.buttonCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_social\_share svg:hover{";
style += "fill: " + settings.buttonActiveCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_playlist\_item,";
style += "\#" + id + " .wptpa\_playlist\_scroll\_handle {";
style += "background: " + settings.playlistBG + ";";
style += "color: " + settings.playlistTextCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_playlist\_item:hover,";
style += "\#" + id + " .wptpa\_playlist\_item.active{";
style += "background: " + settings.playlistCurBG + " !important;";
style += "color: " + settings.playlistTextCLR + " !important;";
style += "}";
style += "\#" + id + " .wptpa\_playlist\_item .wptpa\_marker {";
style += "stroke: " + settings.playlistTextCLR + ";";
style += "}";
style += "\#" + id + " .wptpa\_stat\_icon\_sng,";
style += "\#" + id + " .wptpa\_buy svg,";
style += "\#" + id + " .wptpa\_download svg {";
style += "fill: " + settings.playlistTextCLR + ";";
style += "stroke: " + settings.playlistTextCLR + ";";
style += "}";

style += '';
$("head").append(style);

// Формирируем код плеера
$code = ''; // конейнер для аудио, Сразу вставляем первый трек
// Если, включен компакт режим присваеваем основному контейнеру его класс
if (settings.compactMode) {
$tPlayer.addClass('wptpa\_compact');
}
$code += '';
$code += '';
$code += '';
$code += '

';
// в цикле вставляем обложки в слайдер
for (i = 0; i < playlist.length; i++) {
$code += '
* ![](https://the-grid-user-content.s3-us-west-2.amazonaws.com/%27%20+%20playlist%5Bi%5D.cover%20+%20%27)';
}
$code += '';
$code += '';
$code += '';
$code += '';
$code += '';
$code += '';
$code += '';
$code += '';
$code += '';
$code += '';
$code += '';
$code += '';
$code += '';
$code += '';
$code += '00:0000:00';

// Если треков больше одного - выводим кнопки prev, repeat, next
if (playlist.length \> 1) {
$code += '';
$code += '';
$code += '';
}
// Если треков больше одного и опция shuffle разрешена - выводим ее кнопку
if (playlist.length \> 1 && settings.shuffle) {
$code += '';
}
// Если треков больше одного и опция share разрешена - выводим ее кнопку
if (settings.share) {
$code += '';
}
$code += '';
$code += '';
// Если треков больше одного выводим кнопку toggle playlist, если трек один, в зависимости от значения опционального типа выводим - либо кнопку скачать, либо купить, либо заглушку для верстки
if (playlist.length \> 1) {
$code += '';
} else {

if (!playlist\[0\].dwn) {
playlist\[0\].dwn = "";
}
if (!playlist\[0\].buy) {