<template>
  <div class="contorller"
       :style="{height:playHeight, backgroundColor:setColor, color:iconColor, willChange: Transform}"
       @click="() => {if(!isDisplay) Display()}">
    <div class="iconfont icon-down" v-show="isDisplay" @click.stop="Display"></div>
    <div class="iconfont icon-menu" v-show="isDisplay" @click.stop="caidan"></div>
    <div class="img clearfix"
         :class="{imgPlay: isDisplay}"
         :style="{background: Image}">
      <div class="img-back"
           :style="{background:`linear-gradient(${isDisplay ? setColor : 'to right'} ,transparent ,${setColor})`}"
           @touchstart.prevent="touchStart"
           @touchmove.prevent="touchMove"
           @touchend.prevent="touchEnd"></div>
    </div>
    <div class="title" :class="{'title-play':isDisplay && !isFullLyric}">
      <p class="gequ" v-html="Music.name || '轻听'"></p>
      <p class="geshou" v-html="singerName"></p>
    </div>
    <!-- 歌词区块 -->
    <transition name="fade">
    <div class="fullGeci" :style="{backgroundColor: gecibackColor, color: geciColor}" v-show="isFullLyric && isDisplay"  @click="togglefull">
      <scroll class="ly-wrapper" ref="lyricList" :data="currentLyric && currentLyric.lines">
        <div style="width: 80%;margin: 0 auto;overflow: hidden;">
          <div v-if="currentLyric">
            <p ref="lyricLine" 
               v-for="(line, index) in currentLyric.lines"
               :key='index' 
               :class="{[Current] : currentLineNum === index}" 
               @click.stop="ToLyric(line.time, index)">{{ line.txt }}</p>
          </div>
        </div>
      </scroll>
    </div>
  </transition>
    <div class="geci" v-show="isDisplay && !isFullLyric" @click="togglefull">
      <p>{{ playingLyric }}</p>
    </div>
    <!-- 播放控制区块 -->
      <div class="contorl" :class="{'control-play':isDisplay}">
        <span class="iconfont"
        :class="[modeIndex === 0 ? Mode[modeIndex] : modeIndex === 1 ? Mode[modeIndex] : Mode[modeIndex]]"
        @click.stop="setLoop" v-show="isDisplay"></span>
        <span class="iconfont icon-pre" @click.stop="pre" v-show="isDisplay"></span>
        <span :class="[isPlaying ? unplayClass : playClass]" class="iconfont" @click.stop="ready"></span>
        <span class="iconfont icon-next"  @click.stop="next"  v-show="isDisplay"></span>
        <span :class="[islove ? loveClass : unloveClass]" class="iconfont" @click="Love(Music)" v-show="isDisplay"></span>
        <span class="iconfont icon-list"  @click.stop="showlist" v-show="!isDisplay"></span>
      </div>
    <audio id="media" 
           :src=riceMusicList[riceIndex] 
           ref="audio" 
           @loadstart="playMusic" 
           @timeupdate="updateTime" 
           @ended="next" 
           @loadedmetadata="getLyric" 
           :loop="modeIndex === 0" @error="Error"></audio>
    <div class="progressBar" ref="progressBar">
      <div class="progress" ref="progress"></div>
    </div>
  </div>
</template>
<script>
import { mapGetters, mapMutations, mapActions } from "vuex";
import Scroll from "../../base/scroll.vue";
import popup from "../popup/popup.vue";
import { getLyric } from "../../api/song.js";
import { getMusicVkey } from "../../api/search";
import { Base64 } from "js-base64";
import Lyric from "lyric-parser";
import "../../base/rgbaster.js";

export default {
  components: {
    Scroll,
    popup
  },
  data() {
    return {
      Transform: "auto",
      playHeight: "59px",
      isFullLyric: false,
      islove: false,
      gecibackColor: `rgba(196,176,152,0.9)`,
      setColor: "rgb(196,176,152)",
      iconColor: "#000",
      geciColor: "#000",
      Mode: ["icon-loop", "icon-randoms", "icon-unloop"],
      modeIndex: 2,
      playClass: "icon-play",
      unplayClass: "icon-unplay",
      loveClass: "icon-hongxin",
      unloveClass: "icon-aixin",
      currentTime: 0,
      currentLyric: null,
      currentLineNum: 0,
      playingLyric: "",
      Current: "currentw",
      riceMusicList:[
        "http://www.rice666.com:8888/music/1001.哒哒 - 吃吧，没人嫌你胖.mp3               ",
        "http://www.rice666.com:8888/music/1002.艾热,李佳隆 - 星球坠落 (Live).mp3          ",
        "http://www.rice666.com:8888/music/1003.大贺兄弟 - 痴情玫瑰花.mp3                 ",
        "http://www.rice666.com:8888/music/1004.董又霖 - 就喜欢你.mp3                        ",
        "http://www.rice666.com:8888/music/1005.高宇Slient - 失控.mp3                           ",
        "http://www.rice666.com:8888/music/1006.韩宝仪-舞女泪.mp3                             ",
        "http://www.rice666.com:8888/music/1007.可不可以给我你的微信.mp3                  ",
        "http://www.rice666.com:8888/music/1008.前男友-童话里有什么.mp3                    ",
        "http://www.rice666.com:8888/music/1009.声入人心男团-鹿 Be Free.mp3                  ",
        "http://www.rice666.com:8888/music/1010.孙盛希 - 少一点天分.mp3                    ",
        "http://www.rice666.com:8888/music/1011.祁隆-如果有一天我们都老了.mp3          ",
        "http://www.rice666.com:8888/music/1012.满舒克 - 每一天.mp3                           ",
        "http://www.rice666.com:8888/music/1013.李佳文 - I Wanna.mp3                             ",
        "http://www.rice666.com:8888/music/1014.潘玮柏_弦子-不得不爱.mp3                  ",
        "http://www.rice666.com:8888/music/1015.蒙安 - 甜心.mp3                                 ",
        "http://www.rice666.com:8888/music/1016.娜美 - 醉仙美.mp3                              ",
        "http://www.rice666.com:8888/music/1017.BLACKPINK - BOOMBAYAH (Japanese Ver.).mp3           ",
        "http://www.rice666.com:8888/music/1018.Dawin_Rock City-Bikini Body.mp3                     ",
        "http://www.rice666.com:8888/music/1020.Ed Sheeran-Shape of You.mp3                         ",
        "http://www.rice666.com:8888/music/1023.苏冬涛,昊林 - i love Poland.mp3                ",
        "http://www.rice666.com:8888/music/1024.曲婉婷-Jar Of Love.mp3                           ",
        "http://www.rice666.com:8888/music/1026.曲木陈呷 - Faded（Remix）.mp3                 ",
        "http://www.rice666.com:8888/music/1029.ラムジ - PLANET.mp3                              ",
        "http://www.rice666.com:8888/music/1030.双笙 - 心做し（Cover GUMI）.mp3             ",
        "http://www.rice666.com:8888/music/1031.起风了-买辣椒也用券.mp3                   ",
        "http://www.rice666.com:8888/music/1032.沈以诚 - 独居男子.mp3                       ",
        "http://www.rice666.com:8888/music/1033.田丹 - 相互放手.mp3                           ",
        "http://www.rice666.com:8888/music/1035.汪苏泷 - 忽而今夏.mp3                       ",
        "http://www.rice666.com:8888/music/1037.A-Lin,王晰 - 生活该有的样子.mp3           ",
        "http://www.rice666.com:8888/music/1038.12zzzzzzzz - 棋局.mp3                            ",
        "http://www.rice666.com:8888/music/1039.Nigel Silin - Sakura Tears.mp3                      ",
        "http://www.rice666.com:8888/music/1040.张杰-PERFUME.mp3                                  ",
        "http://www.rice666.com:8888/music/1041.于子将 - 时间走了.mp3                       ",
        "http://www.rice666.com:8888/music/1042.Matteo - Panama.mp3                                 ",
        "http://www.rice666.com:8888/music/1043.Markus Becker-Bum Bum Bum.mp3                       ",
        "http://www.rice666.com:8888/music/1044.周传雄-冬天的秘密.mp3                      ",
        "http://www.rice666.com:8888/music/1045.张芸京-偏爱.mp3                                ",
        "http://www.rice666.com:8888/music/1046.张佑方 - 后来的我们.mp3                    ",
        "http://www.rice666.com:8888/music/1047.张雪飞 - 走心小卖家.mp3                     ",
        "http://www.rice666.com:8888/music/1049.MKJ - Time (Official).mp3                          ",
        "http://www.rice666.com:8888/music/1050.OneRepublic-Counting Stars.mp3                     ",
        "http://www.rice666.com:8888/music/1051.庄心妍-放过自己.mp3                         ",
        "http://www.rice666.com:8888/music/1053.徐秉龙 - 宝贝.mp3                              ",
        "http://www.rice666.com:8888/music/1054.谢军-山谷里的思念.mp3                       ",
        "http://www.rice666.com:8888/music/1055.小星星Aurora - 雨前茶.mp3                     ",
        "http://www.rice666.com:8888/music/1056.赵小臭-未闻花名.mp3                         ",
        "http://www.rice666.com:8888/music/1057.Lola Coca - GQ.mp3                                  ",
        "http://www.rice666.com:8888/music/1058.Nate Dogg - The Next Episode.mp3                    ",
        "http://www.rice666.com:8888/music/1059.NINEONE - I don't wanna see u anymore.mp3           ",
        "http://www.rice666.com:8888/music/1060.OMFG - Hello.mp3                                    ",
        "http://www.rice666.com:8888/music/1061.萧敬腾-怎么说我不爱你.mp3                ",
        "http://www.rice666.com:8888/music/1062.Madcon_Marcus And Martinus-Girls.mp3                ",
        "http://www.rice666.com:8888/music/1063.M2,Miani - Tu Vivi Nell'Aria.mp3                   ",
        "http://www.rice666.com:8888/music/1065.邬真理,YOUNG OTK,池林 - 可爱多.mp3          ",
        "http://www.rice666.com:8888/music/1066.吾尊 - 当你走了.mp3                           ",
        "http://www.rice666.com:8888/music/1067.Maroon 5-Animals.mp3                                ",
        "http://www.rice666.com:8888/music/1068.Niykee Heaton - Skin Tight.mp3                      ",
        "http://www.rice666.com:8888/music/1069.R3hab,Deorro - Flashlight (IR Edit).mp3             ",
        "http://www.rice666.com:8888/music/1071.RT - Umbrella(Meow Remix).mp3                       ",
        "http://www.rice666.com:8888/music/1072.Safia-Counting Sheep.mp3                            ",
        "http://www.rice666.com:8888/music/1073.Sand - CHINA-2.mp3                                  ",
        "http://www.rice666.com:8888/music/1074.Selena Gomez_marshmell o-Wolves.mp3                 ",
        "http://www.rice666.com:8888/music/1075.Psy-Gentleman.mp3                                   ",
        "http://www.rice666.com:8888/music/1076.Pixie Paris-Es Rappelt Im Karton.mp3                ",
        "http://www.rice666.com:8888/music/1078.吴青峰-窗外的天气.mp3                      ",
        "http://www.rice666.com:8888/music/1079.吴哈哈 - 毕业礼.mp3                           ",
        "http://www.rice666.com:8888/music/1080.吴正潇-最后我们没有在一起.mp3          ",
        "http://www.rice666.com:8888/music/1081.夏婉安-失去之后.mp3                          ",
        "http://www.rice666.com:8888/music/1082.萧敬腾-王妃.mp3                                ",
        "http://www.rice666.com:8888/music/1083.未来式(Wills)乐队 - 烟雨江南.mp3          ",
        "http://www.rice666.com:8888/music/1084.李子阳 - 荔枝糖的味道.mp3                 ",
        "http://www.rice666.com:8888/music/1085.董雯兮r - 那天.mp3                             ",
        "http://www.rice666.com:8888/music/1086.大白橙子 - 是醉是醒.mp3                     ",
        "http://www.rice666.com:8888/music/1087.蓝心 - 还偷偷爱你.mp3                       ",
        "http://www.rice666.com:8888/music/1088.泥鳅 - 天文学家.mp3                           ",
        "http://www.rice666.com:8888/music/1089.石页 - 少年和他给大海的歌.mp3           ",
        "http://www.rice666.com:8888/music/1090.十七个远方 - 雪里的油画.mp3              ",
        "http://www.rice666.com:8888/music/1091.刘倩彤Titons - 无花.mp3                       ",
        "http://www.rice666.com:8888/music/1092.蔡程昱 - 那就这样吧.mp3                    ",
        "http://www.rice666.com:8888/music/1093.姜晗_w小婉君-灵魂伴侣.mp3                 ",
        "http://www.rice666.com:8888/music/1094.茶理理 - 童话世界.mp3                       ",
        "http://www.rice666.com:8888/music/1095.白举纲 - 一拳 (Unplugged).mp3                  ",
        "http://www.rice666.com:8888/music/1096.解忧邵帅 - 暖一杯茶.mp3                     ",
        "http://www.rice666.com:8888/music/1097.李泽垚_成泡泡-给你比颗小心心.mp3       ",
        "http://www.rice666.com:8888/music/1098.孟想-这么晚还没睡.mp3                       ",
        "http://www.rice666.com:8888/music/1099.圈圈宝贝-OKOK.mp3                               ",
        "http://www.rice666.com:8888/music/1100.汪昱名-单人旁.mp3                            ",
        "http://www.rice666.com:8888/music/1101.廖俊涛 - 无言以说.mp3                       ",
        "http://www.rice666.com:8888/music/1102.汪苏泷 - 耿.mp3                                ",
        "http://www.rice666.com:8888/music/1103.派克特（PACT） - Sad Man.mp3                   ",
        "http://www.rice666.com:8888/music/1104.橙翼 - 故梦.mp3                                 ",
        "http://www.rice666.com:8888/music/1105.方至圆 - Selina.mp3                              ",
        "http://www.rice666.com:8888/music/1106.江苹果-等风也等你.mp3                       ",
        "http://www.rice666.com:8888/music/1107.Babystop_山竹 - 此歌名遗忘.mp3              ",
        "http://www.rice666.com:8888/music/1108.梁雨-翅膀.mp3                                  ",
        "http://www.rice666.com:8888/music/1109.郭聪明-往南.mp3                                ",
        "http://www.rice666.com:8888/music/1110.刘莱斯 - 庸人.mp3                             ",
        "http://www.rice666.com:8888/music/1111.张燕峰 - 轻童话.mp3                           ",
        "http://www.rice666.com:8888/music/1112.小峰峰_小潘潘-怀念那时候.mp3             ",
        "http://www.rice666.com:8888/music/1113.小星星Aurora-画一个星星一个你.mp3        ",
        "http://www.rice666.com:8888/music/1114.谢雨维 - 光阴失措.mp3                        ",
        "http://www.rice666.com:8888/music/1115.徐良-盗御马(录音室版).mp3                  ",
        "http://www.rice666.com:8888/music/1116.晃儿 - 隔烟水.mp3                             ",
        "http://www.rice666.com:8888/music/1118.陈小熊-济南济南.mp3                          ",
        "http://www.rice666.com:8888/music/1119.刘贞岑-可不可以坐这里.mp3                 ",
        "http://www.rice666.com:8888/music/1120.牧羊人乐队-乖乖你不要撒娇.mp3           ",
        "http://www.rice666.com:8888/music/1121.一只庄哈哈 - 猫.mp3                           ",
        "http://www.rice666.com:8888/music/1122.安子与九妹-迷途的羔羊.mp3                 ",
        "http://www.rice666.com:8888/music/1123.郁可唯-路过人间.mp3                          ",
        "http://www.rice666.com:8888/music/1124.戚勋凯 - 牡丹亭.mp3                           ",
        "http://www.rice666.com:8888/music/1125.戴佩妮-你要的爱.mp3                          ",
        "http://www.rice666.com:8888/music/1126.腔调组合 - 你在哪.mp3                       ",
        "http://www.rice666.com:8888/music/1127.不才,陈鹏杰 - 权香调.mp3                    ",
        "http://www.rice666.com:8888/music/1128.王欣宇-1234喜欢你.mp3                         ",
        "http://www.rice666.com:8888/music/1129.牛奶白 - 06 等我回家 Home.mp3              ",
        "http://www.rice666.com:8888/music/1131.吴佳煜-Say GoodBye.mp3                           ",
        "http://www.rice666.com:8888/music/1132.薛廷佑-爱你是你还是你.mp3                 ",
        "http://www.rice666.com:8888/music/1133.薇小薇 - 半途而废的花.mp3                  ",
        "http://www.rice666.com:8888/music/1134.张蔷,jiafeng - 蹦迪治大病.mp3               ",
        "http://www.rice666.com:8888/music/1135.阿紫 - 别走好吗.mp3                           ",
        "http://www.rice666.com:8888/music/1136.安子与九妹 - 布谷鸟.mp3                     ",
        "http://www.rice666.com:8888/music/1137.周笔畅 - 大象之歌.mp3                        ",
        "http://www.rice666.com:8888/music/1138.秋秋-打电话给你.mp3                          ",
        "http://www.rice666.com:8888/music/1139.焦迈奇 - 超大只怪兽.mp3                     ",
        "http://www.rice666.com:8888/music/1140.任舒瞳_孙羽幽-除了吃就是睡.mp3          ",
        "http://www.rice666.com:8888/music/1141.二茉-奶茶加糖.mp3                             ",
        "http://www.rice666.com:8888/music/1142.安久Angela - 你.mp3                              ",
        "http://www.rice666.com:8888/music/1143.Viola巧克 - 念念渐远.mp3                     ",
        "http://www.rice666.com:8888/music/1144.郑帅 - 人昔小夜曲.mp3                       ",
        "http://www.rice666.com:8888/music/1145.林子初Lynn - 日出的回音.mp3                 ",
        "http://www.rice666.com:8888/music/1146.音阙诗听,清弄 - 食色.mp3                   ",
        "http://www.rice666.com:8888/music/1147.盖巧-塑料姐妹.mp3                             ",
        "http://www.rice666.com:8888/music/1148.萝莉大大-甜心少女.mp3                       ",
        "http://www.rice666.com:8888/music/1149.汪昱名-微爱(Love on the Cloud).mp3             ",
        "http://www.rice666.com:8888/music/1150.kiki今日七分甜 - 为难.mp3                   ",
        "http://www.rice666.com:8888/music/1151.莱恩斯考特RyanScooter-未来的你.mp3         ",
        "http://www.rice666.com:8888/music/1152.冉小冉-我超甜.mp3                             ",
        "http://www.rice666.com:8888/music/1153.周歌歌-我想和你谈恋爱.mp3                 ",
        "http://www.rice666.com:8888/music/1154.小满-无我无他无骨无花.mp3                 ",
        "http://www.rice666.com:8888/music/1155.阿哲-习惯了.mp3                                ",
        "http://www.rice666.com:8888/music/1156.郭诚（橙子） - 我在风里等你.mp3         ",
        "http://www.rice666.com:8888/music/1157.林东-心声.mp3                                   ",
        "http://www.rice666.com:8888/music/1158.文文小婧-小羊羔.mp3                          ",
        "http://www.rice666.com:8888/music/1159.薛禹舟 - 圆沙小曲.mp3                       ",
        "http://www.rice666.com:8888/music/1160..谈丽君 - 余下的时光.mp3                   ",
        "http://www.rice666.com:8888/music/1161.大瑞D.rui - 游荡.mp3                           ",
        "http://www.rice666.com:8888/music/1162.萧煌奇 - 一笔勾销.mp3                       ",
        "http://www.rice666.com:8888/music/1163.傻子与白痴 - 夜长梦少.mp3                 ",
        "http://www.rice666.com:8888/music/1164.筱倩-烟花落.mp3                               ",
        "http://www.rice666.com:8888/music/1165.曾经艺也 - 荀彧.mp3                          ",
        "http://www.rice666.com:8888/music/1166.张津涤_倪可-有你就好.mp3                   ",
        "http://www.rice666.com:8888/music/1167.Faye小糖人 - 怎么会爱上你.mp3             ",
        "http://www.rice666.com:8888/music/1168.陆政廷Lil Jet-自我介绍(Live).mp3             ",
        "http://www.rice666.com:8888/music/1170.小旭音乐,陈雪凝 - 我叫奶瓶.mp3          ",
        "http://www.rice666.com:8888/music/1171.栗先达,陈雪凝 - 没忘.mp3                   ",
        "http://www.rice666.com:8888/music/1172.花粥 - 老郭.mp3                                 ",
        "http://www.rice666.com:8888/music/1173.花粥 - 何苦来哉.mp3                          ",
        "http://www.rice666.com:8888/music/1174.花粥 - 二十岁的某一天.mp3                  ",
        "http://www.rice666.com:8888/music/1175.花粥 - 动物园.mp3                             ",
        "http://www.rice666.com:8888/music/1176.隔壁老樊 - 游.mp3                              ",
        "http://www.rice666.com:8888/music/1178.隔壁老樊 - 四块五.mp3                      ",
        "http://www.rice666.com:8888/music/1181.隔壁老樊 - 你的姑娘.mp3                   ",
        "http://www.rice666.com:8888/music/1182.隔壁老樊 - 男孩（Cover 梁博）.mp3       ",
        "http://www.rice666.com:8888/music/1184.隔壁老樊 - 幻听（Cover：许嵩）.mp3     ",
        "http://www.rice666.com:8888/music/1187.陈雪凝 - 愚昧.mp3                            ",
        "http://www.rice666.com:8888/music/1188.陈雪凝 - 余温.mp3                             ",
        "http://www.rice666.com:8888/music/1189.陈雪凝 - 一口.mp3                            ",
        "http://www.rice666.com:8888/music/1190.陈雪凝 - 无法直视.mp3                       ",
        "http://www.rice666.com:8888/music/1191.陈雪凝 - 我唯一青春里的路人.mp3        ",
        "http://www.rice666.com:8888/music/1192.陈雪凝 - 清守.mp3                             ",
        "http://www.rice666.com:8888/music/1193.陈雪凝 - 你好前任.mp3                       ",
        "http://www.rice666.com:8888/music/1194.陈雪凝 - 那个傻瓜.mp3                      ",
        "http://www.rice666.com:8888/music/1195.陈雪凝 - 没有人心疼我.mp3                ",
        "http://www.rice666.com:8888/music/1196.陈雪凝 - 没那么喜欢.mp3                   ",
        "http://www.rice666.com:8888/music/1197.陈雪凝 - 回忆见.mp3                         ",
        "http://www.rice666.com:8888/music/1198.陈雪凝 - 弹琴给谁听.mp3                    ",
        "http://www.rice666.com:8888/music/1199.陈雪凝 - 避脸.mp3                            ",    
      ],
      riceIndex: 0,
    };
  },
  watch: {
    // 进度条
    currentTime(newcurrent) {
      const all = this.$refs.audio.duration;
      const percent = newcurrent / all;
      const barWidth = this.$refs.progressBar.clientWidth;
      if (percent >= 0) {
        const offsetWidth = barWidth * percent;
        this.$refs.progress.style.width = `${offsetWidth}px`;
      }
    }
  },
  computed: {
    ...mapGetters([
      "Music",
      "isPlaying",
      "isDisplay",
      "loveMusic",
      "currentList",
      "Image"
    ]),
    singerName() {
      this.islove = false;
      this.isLove(this.Music.id).then(res => {
        this.islove = res;
      });
      return this.Music.singer.name;
    }
  },
  created() {
    this.touch = {};
  },
  methods: {
    ...mapMutations([
      "setdialogMsg",
      "delOld",
      "setDisplay",
      "isplay",
      "audioDom",
      "playMusic",
      "addOld",
      "selectmusic"
    ]),
    ...mapActions(["Love", "isLove", "diaShow", "clickPlay"]),
    showlist() {
      if (!this.Music.url) {
        return;
      }
      this.$emit("showLists");
    },
    playMusic(){
      this.$refs.audio.play();
    },
    // 获取图片主题色
    getImageColor() {
      const that = this;
      const host = location.host;
      // canvas不允许获取跨域资源的数据，利用服务器代理的方法，解决跨域问题。
      const URl = `http://${host}/api/img?0=${this.Music.image}`;
      // 把图片绘入canvas利用getImageData获取主题色
      this.Transform = "transform";
      RGBaster.colors(URl, {
        // 调色板大小
        paletteSize: 50,
        // 颜色排除
        exclude: ["rgb(255,255,255)", "rgb(0,0,0)"],
        success: function(payload) {
          // 设置背景色
          that.setColor = payload.dominant;
          // 提取颜色R、G、B值，设置歌词背景颜色
          let c = payload.dominant.match(/\d+/g);
          that.gecibackColor = `rgba(${c[0]},${c[1]},${c[2]},0.8)`;
          // 转换成灰度值判断颜色深浅
          let grayLevel = c[0] * 0.299 + c[1] * 0.587 + c[2] * 0.114;
          if (grayLevel >= 192) {
            // 若为主题色为浅色，把图标颜色设置为黑色
            that.iconColor = "#000";
            // 歌词颜色
            that.geciColor = "rgb(99,118,117)";
            // 歌词高亮为白色
            that.Current = "currentb";
          } else {
            that.iconColor = "#fff";
            that.geciColor = "rgb(219, 182, 182)";
            that.Current = "currentw";
          }
          that.Transform = "auto";
        }
      });
    },
    // 音乐无法加载时触发
    Error() {
      if (!this.Music.url) {
        return;
      }
      this.setdialogMsg("无法播放，已跳过");
      this.diaShow();
      this.$nextTick(() => {
        this.delOld(this.Music);
        this.next();
      });
    },
    // 右上角菜单键
    caidan() {
      if (!this.Music.url) {
        return;
      }
      this.selectmusic(this.Music);
    },
    // 展开
    Display() {
      this.Transform = "transform"
      setTimeout(() => {
        this.Transform = "auto";
      }, 500);
      if(this.isDisplay) {
        this.playHeight = "59px"; 
        this.setDisplay(false);
      } else {
        this.playHeight = "100%";
        this.setDisplay(true);
      }
    },
    // 歌词页面大小切换
    togglefull() {
      if (!this.Music.url) {
        return;
      }
      this.isFullLyric = !this.isFullLyric;
      this.$nextTick(() => {
        this.$refs.lyricList.refresh();
        if (this.currentLineNum > 5) {
          let lineEl = this.$refs.lyricLine[this.currentLineNum - 5];
          this.$refs.lyricList.scrollToElement(lineEl, 600);
        } else {
          this.$refs.lyricList.scrollTo(0, 0, 1000);
        }
      });
    },
    // 切换播放状态
    ready() {
      // if (!this.Music.url) {
      //   return;
      // }
      if (!this.isPlaying) {
        this.$refs.audio.play();
        this.isplay(true);
      } else {
        this.$refs.audio.pause();
        this.isplay(false);
      }
      // if (this.currentLyric) {
      //   this.currentLyric.togglePlay();
      // }
    },
    // 获取歌词
    getLyric() {
      if (!this.Music.mid) {
        return;
      }
      this.getImageColor();
      getLyric(this.Music.mid)
        .then(res => {
          if (res.retcode === 0) {
            return Base64.decode(res.lyric);
          }
        })
        .then(lyric => {
          if (this.currentLyric) {
            this.currentLyric.stop();
          }
          this.currentLyric = new Lyric(lyric, this.handleLyric);
          this.$nextTick(() => {
            if (this.isPlaying) {
              this.currentLyric.play();
            }
          });
          // this.$refs.audio.play();
        });
    },
    // 处理歌词
    handleLyric({ lineNum, txt }) {
      this.currentLineNum = lineNum;
      if (this.currentLineNum > 6) {
        let lineEl = this.$refs.lyricLine[this.currentLineNum - 5];
        this.$refs.lyricList.scrollToElement(lineEl, 600);
      } else {
        this.$refs.lyricList.scrollTo(0, 0, 1000);
      }
      this.playingLyric = txt;
    },
    // 点击歌词跳转到指定位置播放
    ToLyric(time, index) {
      this.$refs.audio.currentTime = time / 1000;
      this.currentLyric.seek(time);
      if(!this.isPlaying) this.ready();
    },
    updateTime(e) {
      this.currentTime = e.target.currentTime;
    },
    // 获取audio传入find组件实现点击播放，移动端不支持autoplay
    getAudio() {
      const au = this.$refs.audio;
      this.audioDom(au);
    },
    // 滑动切歌
    touchStart(e) {
      this.touch.initiated = true;
      const touch = e.touches[0];
      this.touch.startX = touch.pageX;
      this.touch.startY = touch.pageY;
    },
    touchMove(e) {
      if (!this.touch.initiated) {
        return;
      }
      const touch = e.touches[0];
      this.touch.deltaX = touch.pageX - this.touch.startX;
      this.touch.deltaY = touch.pageY - this.touch.startY;
    },
    touchEnd(e) {
      if (this.touch.deltaX > 0 && Math.abs(this.touch.deltaX) > 70) {
        this.touch.deltaX = 0;
        this.pre();
      }
      if (this.touch.deltaX < 0 && Math.abs(this.touch.deltaX) > 70) {
        this.touch.deltaX = 0;
        this.next();
      }
      if (this.touch.deltaY > 0 && Math.abs(this.touch.deltaY) > 70) {
        this.touch.deltaY = 0;
        this.Display();
      } else {
        return;
      }
      this.touch.initiated = false;
    },
    // 下一首
    next() {
      this.riceIndex ++ ;
      if(this.riceIndex >= this.riceMusicList.length){
        this.riceIndex = 0;
      }
      // if (!this.Music.url) {
      //   return;
      // }
      // const len = this.currentList.length;
      // let index = this.Music.index + 1;
      // if (index === len) {
      //   index = 0;
      // }
      // if (this.modeIndex === 1) {
      //   index = Math.floor(Math.random() * len);
      // }
      // this.clickPlay({ item: this.currentList[index], index });
    },
    // 上一首
    pre() {
      this.riceIndex --;
      if(this.riceIndex< 0){
        this.riceIndex = this.riceMusicList.length - 1;
      }
      // if (!this.Music.url) {
      //   return;
      // }
      // let index;
      // const len = this.currentList.length;
      // if (this.Music.index === 0) {
      //   index = len - 1;
      // } else {
      //   index = this.Music.index - 1;
      // }
      // this.clickPlay({ item: this.currentList[index], index });
    },
    // 循环
    setLoop() {
      if (!this.Music.url) {
        return;
      }
      this.modeIndex++;
      if (this.modeIndex === 3) {
        this.modeIndex = 0;
      }
      if (this.modeIndex === 0) {
        this.setdialogMsg("单曲循环");
      } else if (this.modeIndex === 1) {
        this.setdialogMsg("随机播放");
      } else {
        this.setdialogMsg("列表循环");
      }
      this.diaShow();
    }
  }
};
</script>
  <style scoped>
@import "./font_play/iconfont.css";
.contorller {
  width: 100%;
  position: fixed;
  bottom: 0;
  left: 0;
  z-index: 20;
  box-shadow: 3px 0px 4px 0 rgba(167, 135, 135, 0.7);
  transition: all 0.4s cubic-bezier(0.15, 0.65, 0.35, 0.97);
}

.icon-menu {
  position: absolute;
  top: 0;
  right: 0;
  margin: 15px;
  z-index: 26;
}
.icon-down {
  position: absolute;
  top: 0;
  left: 0;
  margin: 15px;
  z-index: 26;
}
.contorller .img {
  width: 54px;
  height: 54px;
  background-size: cover !important;
  background-position: center center;
  position: relative;
  transition: all 0.4s cubic-bezier(0.15, 0.65, 0.35, 0.97);
}
.img .img-back {
  width: 101%;
  height: 101%;
}
.imgPlay {
  width: 100% !important;
  height: 55% !important;
  transform-origin: top left !important;
}
.title {
  position: absolute;
  top: 0px;
  left: 7em;
  transition: left 0.4s ease;
  z-index: 26;
  font-size: 62.5%;
}
.title .gequ {
  font-size: 0.75rem;
  line-height: 30px;
  text-align: center;
}
.title .geshou {
  font-size: 0.75rem;
  text-align: center;
}
.contorl {
  position: absolute;
  display: flex;
  justify-content: center;
  bottom: 10px;
  right: 20px;
  transition: all 0.4s;
  vertical-align: middle;
  /* 中线对齐 */
  align-items: center;
  z-index: 25;
}
.contorl span + span {
  margin-left: 30px;
}
.title-play {
  top: 62%;
  left: 50%;
  transform: translate3d(-50%, -50%, 0);
}
.control-play {
  width: 100%;
  right: 0;
  bottom: 40px;
  display: flex;
  justify-content: center;
  font-size: 30px;
}
.clearfix:after,
.clearfix:before {
  content: " ";
  display: table;
}
.clearfix:after {
  clear: both;
}
.currentw {
  color: #fff;
}
.currentb {
  color: #000;
}
.progressBar {
  width: 100%;
  height: 5px;
  position: absolute;
  left: 0;
  bottom: 0px;
  background-color: rgb(0, 0, 0);
  z-index: 25;
}
.progress {
  width: 0;
  height: 5px;
  background-color: #fff;
  z-index: 25;
}
.geci {
  width: 100%;
  position: absolute;
  height: 50px;
  bottom: 15%;
  line-height: 20px;
  padding: 50px 10px;
  overflow: hidden;
  box-sizing: border-box;
}
.geci p {
  text-align: center;
  font-size: 13px;
}
.fullGeci {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  z-index: 25;
  line-height: 35px;
}
.fullGeci p {
  text-align: center;
  font-size: 13px;
}
.ly-wrapper {
  margin: 95px 10px 20px 10px;
  left: 0;
  right: 0;
  height: 56%;
  overflow: hidden;
}
</style>
