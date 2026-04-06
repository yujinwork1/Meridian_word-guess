<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1,maximum-scale=1,user-scalable=no,viewport-fit=cover">
<title>Weekly Word Guess</title>
</head>
<body>
<div id="wgApp"></div>
<script>
(function(){
'use strict';

var s=document.createElement('style');
s.textContent=`
@import url('https://fonts.googleapis.com/css2?family=DM+Mono:wght@400;500&family=DM+Sans:wght@300;400;500;600;700&display=swap');
*,*::before,*::after{box-sizing:border-box;margin:0;padding:0;-webkit-tap-highlight-color:transparent}
html{height:100%}
body{background:#08101e;min-height:100vh;overflow-x:hidden}
.wg-wrap{font-family:'DM Sans',sans-serif;background:#08101e;color:#e2e8f0;min-height:100vh;padding-bottom:48px}

/* HEADER */
.wg-header{background:#0a1525;border-bottom:1px solid #1e3a5f;padding:18px 16px 16px;text-align:center}
.wg-ms-logo{display:block;margin:0 auto 12px;height:34px;width:auto;max-width:190px;object-fit:contain}
.wg-title-svg{display:block;margin:0 auto 10px;width:min(96vw,180px);height:auto}
.wg-edition{font-size:10px;letter-spacing:.15em;color:#3b82f6;text-transform:uppercase;font-weight:600;margin-bottom:5px}
.wg-hdate{font-size:14px;font-weight:600;color:#f0f6ff}
.wg-hdate span{color:#3b82f6}

/* CONTROLS */
.wg-controls{padding:14px 14px 0;display:flex;justify-content:center;gap:7px;flex-wrap:wrap}
.wg-btn{font-family:'DM Sans',sans-serif;font-size:12px;font-weight:600;padding:8px 15px;border-radius:8px;border:none;cursor:pointer;transition:all .18s;letter-spacing:.02em;touch-action:manipulation;white-space:nowrap}
.wg-btn-ghost{background:#0f1e33;color:#60a5fa;border:1px solid #1e3a5f}
.wg-btn-ghost:hover,.wg-btn-ghost:active{background:#1e3a5f}
.wg-btn-reset{background:#0f1e33;color:#f87171;border:1px solid #2d1515}
.wg-btn-reset:hover,.wg-btn-reset:active{background:#2d1515}

/* GAME */
.wg-game{display:flex;flex-direction:column;align-items:center;padding:14px 14px 0;gap:12px}

/* STATS */
.wg-stats-row{display:flex;gap:8px;width:100%;max-width:420px}
.wg-sbox{background:#0a1525;border:1px solid #1e3a5f;border-radius:10px;padding:7px 8px;text-align:center;flex:1;min-width:0}
.wg-sbox-lbl{font-size:9px;letter-spacing:.08em;color:#4a6080;text-transform:uppercase;font-weight:600;margin-bottom:2px;white-space:nowrap}
.wg-sbox-val{font-family:'DM Mono',monospace;font-size:clamp(14px,3.5vw,18px);color:#60a5fa;font-weight:500;line-height:1.2}
.wg-sbox-val.h{color:#a78bfa}
.wg-sbox-val.t{color:#4ade80}

/* GRID */
.wg-grid{display:flex;flex-direction:column;gap:6px}
.wg-row{display:flex;gap:6px;justify-content:center}
.wg-tile{font-family:'DM Mono',monospace;font-weight:500;display:flex;align-items:center;justify-content:center;background:#06101c;color:#e2e8f0;border:2px solid #1e3a5f;border-radius:6px;text-transform:uppercase;transition:border-color .05s}
.wg-tile.tbd{border-color:#2563eb}
.wg-tile.pop{animation:wgPop .09s ease-out}
.wg-tile.shake{animation:wgShk .42s ease}
.wg-tile.bounce{animation:wgBnc .65s ease}
.wg-tile.correct{background:#0d2218;border-color:#166534;color:#4ade80}
.wg-tile.present{background:#1c1706;border-color:#92400e;color:#fbbf24}
.wg-tile.absent{background:#111827;border-color:#111827;color:#374151}
@keyframes wgPop{0%{transform:scale(1)}50%{transform:scale(1.13)}100%{transform:scale(1)}}
@keyframes wgShk{0%,100%{transform:translateX(0)}15%{transform:translateX(-5px)}35%{transform:translateX(5px)}55%{transform:translateX(-4px)}75%{transform:translateX(4px)}90%{transform:translateX(-2px)}}
@keyframes wgBnc{0%{transform:translateY(0)}20%{transform:translateY(-28px)}40%{transform:translateY(-6px)}60%{transform:translateY(-16px)}80%{transform:translateY(-4px)}100%{transform:translateY(0)}}
@keyframes wgGlow{0%,100%{box-shadow:0 0 8px rgba(74,222,128,.4)}50%{box-shadow:0 0 24px rgba(74,222,128,.9),0 0 48px rgba(74,222,128,.4)}}
@keyframes wgConfettiFall{0%{transform:translateY(-20px) rotate(0deg);opacity:1}100%{transform:translateY(100vh) rotate(720deg);opacity:0}}
@keyframes wgOverlayIn{0%{opacity:0;transform:scale(.88)}100%{opacity:1;transform:scale(1)}}
@keyframes wgPulse{0%,100%{transform:scale(1)}50%{transform:scale(1.06)}}

/* WIN OVERLAY */
.wg-win-overlay{position:fixed;inset:0;z-index:999;display:flex;align-items:center;justify-content:center;background:rgba(4,8,20,.82);backdrop-filter:blur(6px);-webkit-backdrop-filter:blur(6px)}
.wg-win-box{background:#0a1525;border:2px solid #166534;border-radius:18px;padding:28px 24px 24px;text-align:center;max-width:320px;width:90vw;animation:wgOverlayIn .35s cubic-bezier(.34,1.56,.64,1) forwards;box-shadow:0 0 60px rgba(74,222,128,.25)}
.wg-win-emoji{font-size:52px;margin-bottom:10px;animation:wgPulse 1.2s ease infinite}
.wg-win-title{font-family:'DM Sans',sans-serif;font-size:24px;font-weight:700;color:#4ade80;margin-bottom:4px;letter-spacing:-.01em}
.wg-win-subtitle{font-size:13px;color:#4a6080;margin-bottom:16px}
.wg-win-word{font-family:'DM Mono',monospace;font-size:32px;font-weight:500;color:#f0f6ff;letter-spacing:.18em;margin-bottom:6px;animation:wgGlow 1.5s ease infinite}
.wg-win-meta{font-size:12px;color:#4a6080;margin-bottom:20px}
.wg-win-meta span{color:#60a5fa;font-family:'DM Mono',monospace}
.wg-win-btns{display:flex;flex-direction:column;gap:8px}
.wg-win-submit-row{display:flex;gap:8px;margin-bottom:2px}
.wg-win-name{flex:1;font-family:'DM Sans',sans-serif;font-size:13px;background:#06101c;border:1px solid #1e3a5f;border-radius:8px;padding:10px 12px;color:#e2e8f0;outline:none;transition:border-color .2s;-webkit-appearance:none}
.wg-win-name:focus{border-color:#3b82f6}
.wg-win-name::placeholder{color:#4a6080}
.wg-win-submit{font-family:'DM Sans',sans-serif;font-size:13px;font-weight:600;background:#166534;color:#4ade80;border:1px solid #166534;border-radius:8px;padding:10px 16px;cursor:pointer;white-space:nowrap;transition:all .18s}
.wg-win-submit:hover{background:#15803d}
.wg-win-submit:disabled{opacity:.4;cursor:default}
.wg-win-close{font-family:'DM Sans',sans-serif;font-size:13px;font-weight:600;background:#0f1e33;color:#60a5fa;border:1px solid #1e3a5f;border-radius:8px;padding:10px;cursor:pointer;transition:all .18s}
.wg-win-close:hover{background:#1e3a5f}

/* CONFETTI PIECE */
.wg-confetti{position:fixed;top:-10px;pointer-events:none;z-index:1000;width:8px;height:8px;border-radius:2px;animation:wgConfettiFall linear forwards}

/* HINT CARD */
.wg-hint-card{background:#0a1525;border:1px solid #1e3a5f;border-radius:10px;padding:12px 14px;width:100%;max-width:420px;display:none}
.wg-hint-lbl{font-size:10px;color:#4a6080;letter-spacing:.12em;text-transform:uppercase;font-weight:600;margin-bottom:8px}
.wg-hint-tiles{display:flex;gap:5px;justify-content:center}
.wg-htile{font-family:'DM Mono',monospace;font-size:14px;font-weight:500;width:34px;height:34px;border-radius:5px;border:1px solid #1e3a5f;background:#06101c;color:#4a6080;display:flex;align-items:center;justify-content:center;text-transform:uppercase}
.wg-htile.rev{color:#93c5fd;border-color:#2563eb;background:#0f1e33}

/* KEYBOARD */
.wg-keyboard{display:flex;flex-direction:column;gap:6px;align-items:center;width:100%;max-width:420px}
.wg-kb-row{display:flex;gap:5px;justify-content:center;width:100%}
.wg-key{font-family:'DM Mono',monospace;font-weight:500;border-radius:6px;border:1px solid #1e3a5f;background:#0f1e33;color:#c0cce0;cursor:pointer;display:flex;align-items:center;justify-content:center;transition:background .12s,color .12s;touch-action:manipulation;-webkit-user-select:none;user-select:none;flex:1}
.wg-key.wide{flex:1.6}
.wg-key:hover{background:#1e3a5f}
.wg-key:active{transform:scale(.9)}
.wg-key.correct{background:#0d2218;border-color:#166534;color:#4ade80}
.wg-key.present{background:#1c1706;border-color:#92400e;color:#fbbf24}
.wg-key.absent{background:#0a0f1a;border-color:#0a0f1a;color:#1e2a3a}

/* MSG */
.wg-msg{margin:12px 14px 0;padding:12px 18px;border-radius:10px;font-size:13px;font-weight:600;display:none;text-align:center}
.wg-msg.win{background:#0d2218;border:1px solid #166534;color:#4ade80;display:block}
.wg-msg.over{background:#2d0f0f;border:1px solid #7f1d1d;color:#f87171;display:block}
.wg-msg.info{background:#0f1e33;border:1px solid #1e3a5f;color:#60a5fa;display:block}



/* LEADERBOARD */
.wg-lb-wrap{padding:0 14px;margin-top:16px}
.wg-lb{background:#0a1525;border:1px solid #1e3a5f;border-radius:12px;padding:16px 14px}
.wg-lb-head{display:flex;align-items:center;justify-content:space-between;margin-bottom:14px;flex-wrap:wrap;gap:6px}
.wg-lb-title{font-size:14px;font-weight:700;color:#f0f6ff}
.wg-lb-sub{font-size:11px;color:#4a6080}
.wg-lb-table{width:100%;border-collapse:collapse}
.wg-lb-table th{font-size:10px;color:#4a6080;letter-spacing:.08em;text-transform:uppercase;font-weight:600;padding:0 5px 9px;text-align:left;white-space:nowrap}
.wg-lb-table td{padding:9px 5px;border-top:1px solid #0f1e33;font-size:12px;color:#94a3b8;vertical-align:middle;white-space:nowrap}
.wg-lb-table tr:first-child td{border-top:none}
.wg-lb-table tr:hover td{background:#0d1a2d}
.wg-rank{font-family:'DM Mono',monospace;font-size:14px}
.wg-plr{color:#e2e8f0;font-weight:500}
.wg-tv{font-family:'DM Mono',monospace;color:#60a5fa;font-weight:500}
.wg-sv{font-family:'DM Mono',monospace;color:#4ade80}
.wg-hv{font-family:'DM Mono',monospace;color:#a78bfa}
.wg-dv{font-size:11px;color:#4a6080;font-family:'DM Mono',monospace}
.wg-lb-empty{text-align:center;color:#4a6080;font-size:13px;padding:24px 0}
.wg-lb-loading{text-align:center;color:#4a6080;font-size:13px;padding:20px 0}
.wg-submit{margin-top:14px;padding-top:14px;border-top:1px solid #0f1e33}
.wg-submit-lbl{font-size:10px;color:#4a6080;letter-spacing:.12em;text-transform:uppercase;font-weight:600;margin-bottom:8px}
.wg-submit-row{display:flex;gap:8px}
.wg-name-input{flex:1;font-family:'DM Sans',sans-serif;font-size:13px;background:#06101c;border:1px solid #1e3a5f;border-radius:8px;padding:9px 12px;color:#e2e8f0;outline:none;transition:border-color .2s;-webkit-appearance:none}
.wg-name-input::placeholder{color:#4a6080}
.wg-name-input:focus{border-color:#3b82f6}
.wg-submit-btn{font-family:'DM Sans',sans-serif;font-size:13px;font-weight:600;background:#1d4ed8;color:#fff;border:none;border-radius:8px;padding:9px 16px;cursor:pointer;transition:all .18s;white-space:nowrap;touch-action:manipulation}
.wg-submit-btn:hover{background:#2563eb}
.wg-submit-btn:disabled{opacity:.4;cursor:default}

/* TOAST */
.wg-tw{position:fixed;top:60px;left:50%;transform:translateX(-50%);z-index:9999;display:flex;flex-direction:column;align-items:center;gap:6px;pointer-events:none;width:90vw}
.wg-toast{background:#f0f6ff;color:#08101e;padding:9px 18px;border-radius:6px;font-family:'DM Sans',sans-serif;font-size:13px;font-weight:700;opacity:0;transform:translateY(-8px);transition:opacity .2s,transform .2s;text-align:center;box-shadow:0 4px 20px rgba(0,0,0,.4)}
.wg-toast.show{opacity:1;transform:translateY(0)}


`;
document.head.appendChild(s);
document.head.appendChild(s);

// ── CONFIG ──
var SCRIPT_URL='https://script.google.com/macros/s/AKfycbxIuBgzjNnvF7cW-YhNqiemWRcJaCOCId4qFOErcP3aXAYI7hQ_DYPuwR2N38U15V8Z/exec';
var LOGO_URL='https://meridiansource.ca/wp-content/uploads/2026/03/Meridian-Source-Logo.png';

function getWeekKey(){
  var d=new Date(),day=d.getDay();
  var m=new Date(d);m.setDate(d.getDate()-((day+6)%7));
  return 'wg_week_'+m.getFullYear()+'_'+(m.getMonth()+1)+'_'+m.getDate();
}
var WEEK=getWeekKey();

// ── WORD LIST — strictly 5-letter only ──
var WORDS=[
  "ABYSS","ACUTE","ADEPT","AGILE","ALOFT","AMEND","ARDOR","ARMOR","AROSE","ASTIR",
  "ATONE","BLUNT","BOXER","BRASH","BRAWN","BROIL","BROOD","CADET","CAMEL","CARGO",
  "CARVE","CHALK","CHAMP","CHANT","CHASM","CHESS","CIVIC","CLAMP","CLASP","CLEFT",
  "CLIFF","CLING","CLOAK","CLONE","CLOVE","COMET","CORAL","COVEN","CRAFT","CRANE",
  "CREEK","CRUEL","CRUSH","CRYPT","DEALT","DECAY","DECOY","DELVE","DEPOT","DERBY",
  "DODGE","DRAFT","DRAIN","DRAWL","DREAD","DRIFT","DRILL","DRONE","DRUNK","EBONY",
  "BRAVE","CLOUD","DANCE","EARTH","FLAME","GRACE","HAPPY","IMAGE","JUICE","KNIFE",
  "LIGHT","MAGIC","NIGHT","OCEAN","PEACE","QUEEN","RIVER","SMILE","TIGER","UNITY",
  "VOICE","WATER","YOUTH","ANGEL","BLAZE","CHAIR","DREAM","EAGLE","FAITH","GHOST",
  "HEART","IDEAL","JEWEL","KARMA","LASER","MAPLE","NOBLE","OPTIC","PILOT","QUEST",
  "ROBIN","SUGAR","TASTE","VAULT","WHEAT","CANDY","BLISS","CRISP","DAISY","ENTRY",
  "GLOBE","HANDY","IRONY","KNEEL","LOVER","MAYOR","NOVEL","OUNCE","PLAZA","QUICK",
  "RAINY","SHINY","TRULY","ACORN","BALMY","BIRCH","BISON","BLAND","BLOOM","BONUS",
  "BRINE","BRINK","BRISK","BROAD","BROTH","BROWN","BUDGE","BULGE","BUMPY","BURLY",
  "BURNT","CACHE","CHAOS","CHARM","CHASE","CHEAP","CHEEK","CHILI","CHORD","CINCH",
  "CLOWN","COMMA","COURT","DAUNT","DEBUT","DEFER","DELTA","DENSE","DEPTH","DIRTY",
  "DITTY","DIVOT","DIZZY","DUSTY","DWELT","DYING","ECLAT","ELDER","ELITE","ENACT",
  "EPOCH","EQUIP","EVADE","EXACT","EXILE","EXTOL","FAINT","FANCY","FAULT","FEAST",
  "FELON","FERAL","FIEND","FILTH","FINCH","FLAIR","FLASK","FLESH","FLING","FLINT",
  "FLIRT","FLOCK","FLOOD","FOCAL","FOLLY","FORGE","FORTE","FOUND","FRAIL","FRANK",
  "FRAUD","FREAK","FRESH","FROND","FROST","FROTH","FROZE","FULLY","FUNNY","GAVEL",
  "GIDDY","GIVEN","GLEAM","GLEAN","GLINT","GLOOM","GLOSS","GORGE","GOUGE","GRASP",
  "GRAVE","GRAZE","GREED","GREET","GRIEF","GRILL","GRIMY","GROAN","GROIN","GRUFF",
  "GUILD","GUILE","GUISE","GULCH","GUSTO","GYPSY","HARDY","HARSH","HAVEN","HEIST",
  "HENCE","HORDE","HOVEL","HUMAN","HUMOR","HURRY","INEPT","INERT","INFER","INLAY",
  "INNER","INPUT","INTER","IRATE","IVORY","JAUNT","JOUST","JUDGE","JUICY","JUMPY",
  "JUNTO","KEYED","KNAVE","KNOLL","KNOWN","LARGE","LATCH","LAUGH","LAYER","LEARN",
  "LEGAL","LEMON","LEVEL","LIMIT","LOFTY","LOGIC","LOOSE","LOUSE","LUCID","LUCKY",
  "LUSTY","LYRIC","MANOR","MARCH","MARSH","MATCH","MIRTH","MISER","MIXED","MODEL",
  "MONEY","MONTH","MORAL","MOTTO","MOUNT","MOUTH","MURKY","MUSTY","NAIVE","NIFTY",
  "NOBLE","NOTCH","NOVEL","NURSE","NYMPH","OCCUR","OLIVE","OPTIC","OUGHT","OUTDO",
  "OVARY","OVERT","OXIDE","PADDY","PANSY","PETTY","PHASE","PIANO","PIXEL","PLAID",
  "PLANK","PLANT","PLEAD","PLUCK","PLUMB","POACH","POKER","POLAR","POPPY","POUCH",
  "PRANK","PRAWN","PRESS","PRISM","PRIVY","PRONE","PROWL","PSALM","PUDGY","PULSE",
  "PURGE","QUACK","QUALM","QUART","QUIRK","QUOTA","RABBI","RADAR","RADIX","RALLY",
  "RAVEN","REALM","REBEL","REIGN","RELAX","REPAY","REPEL","RESIN","REVEL","RIGID",
  "RISKY","RIVAL","ROAST","RODEO","ROGUE","ROVER","ROWDY","RUDDY","RUSTY","SADLY",
  "SAINT","SALSA","SANDY","SANER","SASSY","SAVOR","SCALD","SCALP","SCANT","SCARE",
  "SCORN","SCOWL","SCRUB","SEIZE","SERGE","SERVE","SEVEN","SHADY","SHAKY","SHAWL",
  "SHEER","SHELF","SHELL","SHIFT","SHIRK","SHRUB","SIEGE","SIGMA","SISSY","SIXTH",
  "SKIMP","SKULL","SLAIN","SLANG","SLASH","SLEEK","SLEET","SLICK","SLIME","SLINK",
  "SLOPE","SLOTH","SLUGS","SLUMP","SLUNG","SLUNK","SLURP","SMACK","SMEAR","SMELT",
  "SMIRK","SMOCK","SMUDGE","SNARE","SNARL","SNEER","SNIDE","SNIFF","SNORE","SNORT",
  "SOOTH","SORRY","SOUTH","SPARK","SPASM","SPAWN","SPECK","SPILL","SPINE","SPIRE",
  "SPITE","SPLAT","SPOOK","SPORE","SPOUT","SPRIG","SPURN","SQUAT","SQUIB","STALK",
  "STALL","STAMP","STANK","STAVE","STEAK","STEAD","STEAL","STEED","STERN","STIFF",
  "STINK","STOMP","STOOP","STOUT","STOVE","STRAP","STRAW","STRAY","STREP","STRUT",
  "STUMP","STUNG","STUNK","STUNT","STYLE","SUAVE","SULKY","SULLEN","SUMAC","SUNNY",
  "SWAMP","SWEAR","SWEEP","SWEET","SWEPT","SWIFT","SWILL","SWOOP","TABBY","TAFFY",
  "TAPIR","TARDY","TAUNT","TAWNY","TERSE","THEFT","THIGH","THORN","THOSE","THUMP",
  "TIARA","TITHE","TODAY","TOPAZ","TOXIC","TRACE","TRACK","TRAMP","TRAWL","TREAD",
  "TRIPE","TRITE","TROMP","TROTH","TROUT","TRUCE","TRUMP","TRUNK","TRUSS","TUBER",
  "TULIP","TUMOR","TUNIC","TUTOR","TWANG","TWEAK","TWEED","TWICE","TWILL","TWIRL",
  "UDDER","ULCER","ULTRA","UMBRA","UNCLE","UNDUE","UNFIT","UNIFY","UNION","UNTIL",
  "UNWED","UPPER","USURP","UTTER","VAGUE","VALOR","VALVE","VAUNT","VENAL","VERSE",
  "VICAR","VIGIL","VIPER","VISOR","VOMIT","VOUCH","WAGER","WALTZ","WARTY","WEARY",
  "WEAVE","WEDGE","WEEDY","WEIRD","WHACK","WHIFF","WHIRL","WHISK","WHOLE","WHOSE",
  "WINDY","WITCH","WITTY","WRATH","WRECK","WRING","WRONG","WROTE","ZEBRA","ZESTY"
];
var VALID_SET=new Set(WORDS);
// common 5-letter words accepted as guesses
var EXTRA=new Set(["ABOUT","ABOVE","ABUSE","ACTOR","AFTER","AGAIN","AGENT","AGREE","AHEAD","ALARM","ALBUM","ALERT","ALIKE","ALIVE","ALLEY","ALLOW","ALONE","ALONG","ALTER","AMINO","AMONG","ANGER","ANGLE","ANGRY","APPLY","ARGUE","ARISE","ARRAY","ASIDE","ASKED","ASSET","ATLAS","AVOID","AWAKE","AWARD","AWARE","BADLY","BASIC","BASIS","BEACH","BEGAN","BEGIN","BEING","BELOW","BENCH","BIBLE","BIRTH","BLANK","BLAST","BLEED","BLEND","BLESS","BLOCK","BLOOD","BLOWN","BOARD","BOOST","BOOTH","BOUND","BRAIN","BREAK","BREED","BRICK","BRIEF","BRING","BROKE","BROOK","BRUSH","BUILD","BUILT","BUNCH","BURST","BUYER","CABIN","CARRY","CATCH","CAUSE","CEASE","CHAIN","CHART","CHECK","CHEER","CHEST","CHIEF","CHILD","CHINA","CHUNK","CIVIL","CLAIM","CLASH","CLASS","CLEAN","CLEAR","CLERK","CLICK","CLIMB","CLOCK","CLOSE","COAST","COSTS","COUNT","COVER","CRACK","CRASH","CRAZY","CREAM","CRIME","CROSS","CROWD","CROWN","CURVE","CYCLE","DAILY","DAIRY","DATUM","DEALS","DEATH","DELAY","DRAMA","DRAWN","DRIVE","DROVE","DRUGS","DRUMS","EARLY","EIGHT","ELECT","EMAIL","EMPTY","ENEMY","ENJOY","ENTER","EQUAL","ESSAY","EVERY","EXACT","EXIST","EXTRA","FALLS","FALSE","FEWER","FIELD","FIFTH","FIFTY","FIGHT","FINAL","FIRST","FIXED","FLASH","FLOOR","FRAME","FRONT","FRUIT","GIANT","GLASS","GOING","GRANT","GRASS","GREAT","GREEN","GRIND","GROSS","GROUP","GROWN","GUARD","GUESS","GUIDE","GUILT","HANDS","HASTE","HEARD","HEAVY","HEDGE","HERBS","HURLY","INDEX","ISSUE","ITEMS","JAPAN","JOINT","LEMON","LEARN","LEGAL","LEVEL","LIMIT","LOGIC","MONEY","MONTH","MORAL","MOTOR","MOUNT","MOVED","MOVES","MUSIC","NAIVE","NAMED","NASTY","NEEDS","NERVE","NEVER","NEWLY","NORTH","NOTED","NOTES","NUDGE","OCCUR","OFTEN","ONSET","ORDER","OTHER","OUTER","OWNER","PAGES","PAINT","PANIC","PAPER","PARTS","PARTY","PATCH","PAUSE","PHASE","PHONE","PHOTO","PIECE","PIZZA","PLACE","PLAIN","PLANE","PLANT","PLATE","POINT","POSED","POUND","POWER","PRESS","PRICE","PRIDE","PRIME","PRINT","PRIOR","PRIZE","PROBE","PROOF","PROUD","PROVE","PULSE","PUPPY","QUOTE","RAISE","RALLY","RANCH","RANGE","RAPID","RATIO","REACH","READY","REALM","REBEL","REFER","REIGN","RELAX","REPLY","RIDER","RIDGE","RIGHT","RISKY","RIVAL","ROCKY","ROMAN","ROOMS","ROOTS","ROUGH","ROUND","ROYAL","RUGBY","RULER","SADLY","SAINT","SALAD","SALES","SCALE","SCARY","SCENE","SCORE","SCOUT","SEDAN","SEIZE","SENSE","SERVE","SEVEN","SHADE","SHAKE","SHALL","SHAME","SHAPE","SHARP","SHELF","SHELL","SHIFT","SHIRT","SHORT","SIGHT","SINCE","SIXTH","SIXTY","SIZED","SKILL","SLATE","SLAVE","SLEEP","SLICE","SLIDE","SLOPE","SMART","SMELL","SNACK","SNAKE","SOLAR","SOLID","SOLVE","SORRY","SOUTH","SPACE","SPARE","SPARK","SPEAK","SPEND","SPENT","SPLIT","SPOKE","SPOON","SPORT","SPRAY","STACK","STAFF","STAGE","STAIN","STAKE","STAND","STARK","START","STATE","STEEL","STEEP","STERN","STICK","STIFF","STILL","STOCK","STONE","STORM","STORY","STRAP","STRIP","STUCK","STUDY","STUFF","STYLE","SUITE","SUNNY","SUPER","SURGE","SWAMP","SWEAR","SWEEP","SWEET","SWEPT","SWIFT","SWORN","SWORD","TABLE","TAKEN","TALES","TALKS","TEETH","THANK","THEIR","THEME","THERE","THICK","THING","THINK","THIRD","THOSE","THREE","THREW","THROW","TIGHT","TIMES","TIRED","TITLE","TODAY","TOKEN","TOTAL","TOUCH","TOUGH","TOWER","TOWNS","TRACK","TRADE","TRAIL","TRAIN","TRAIT","TRASH","TREAT","TREND","TRIAL","TRIED","TRIES","TROOP","TROUT","TRUCK","TRUTH","TYPES","UNDER","UNION","UNTIL","UPPER","UPSET","URBAN","USAGE","USUAL","VALID","VALUE","VIDEO","VIGOR","VIRAL","VISIT","VITAL","VIVID","VOWED","WASTE","WATCH","WHERE","WHICH","WHILE","WHITE","WHOLE","WHOSE","WIDER","WOODS","WORDS","WORKS","WORLD","WORRY","WORSE","WORST","WORTH","WOULD","WRIST","WRONG","WROTE","YARDS","YEARS","YIELD","YOUNG","YOURS","ZONES"]);

// ── STATE ──
var G={word:'',wordLen:5,maxGuesses:6,hintsLeft:2,hintsUsed:[],guesses:[],results:[],currentInput:'',keyStates:{},timer:0,timerInt:null,finished:false,won:false,score:0,isProcessing:false};
var stats=JSON.parse(localStorage.getItem('wg3_stats')||'{"played":0,"wins":0,"streak":0,"maxStreak":0,"dist":[0,0,0,0,0,0]}');

// ── HELPERS ──
function $(id){return document.getElementById(id)}
function ft(s){var m=Math.floor(s/60),ss=s%60;return(m<10?'0':'')+m+':'+(ss<10?'0':'')+ss}
function esc(x){return String(x).replace(/[&<>"']/g,function(c){return({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;',"'":'&#39;'})[c]})}
function fd(ts){
  if(!ts)return'-';
  var d=new Date(Number(ts));
  if(isNaN(d.getTime()))return'-';
  var mo=d.getMonth()+1, dd=d.getDate(), yy=String(d.getFullYear()).slice(2);
  var hh=d.getHours(), mm=d.getMinutes();
  return mo+'/'+dd+'/'+yy+' '+(hh<10?'0':'')+hh+':'+(mm<10?'0':'')+mm;
}
function dateStr(){
  var D=['Sunday','Monday','Tuesday','Wednesday','Thursday','Friday','Saturday'];
  var M=['January','February','March','April','May','June','July','August','September','October','November','December'];
  var d=new Date();return D[d.getDay()]+', '+M[d.getMonth()]+' '+d.getDate()+', '+d.getFullYear();
}

// ── SIZING ──
function tileSz(){
  var vw=Math.min(window.innerWidth,460);
  var sz=Math.floor((vw-28-(G.wordLen-1)*6)/G.wordLen);
  return Math.min(Math.max(sz,44),68);
}
function keySz(){
  var h=Math.min(Math.max(Math.floor(window.innerHeight*0.064),40),52);
  var fs=Math.round(h*0.28);
  return{h:h,fs:fs};
}
function applySize(){
  var sz=tileSz(),ks=keySz();
  document.querySelectorAll('.wg-tile').forEach(function(t){t.style.width=sz+'px';t.style.height=sz+'px';t.style.fontSize=Math.round(sz*0.42)+'px';});
  // Only update size — do NOT touch className so color classes are preserved
  document.querySelectorAll('.wg-key').forEach(function(k){k.style.height=ks.h+'px';k.style.fontSize=ks.fs+'px';});
}
window.addEventListener('resize',applySize);
window.addEventListener('orientationchange',function(){setTimeout(applySize,160)});

// ── BUILD HTML ──
(function(){
  var app=$('wgApp');
  app.className='wg-wrap';
  app.innerHTML=`
<div class="wg-tw" id="wgTW"></div>
<div class="wg-header">
  <img class="wg-ms-logo" src="${LOGO_URL}" alt="Meridian Source" onerror="this.style.display='none'">
  <svg class="wg-title-svg" viewBox="0 0 160 78" xmlns="http://www.w3.org/2000/svg">
    <text x="80" y="22" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="22" font-weight="300" fill="#4a6080">Weekly</text>
    <text x="80" y="50" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="22" font-weight="700" fill="#f0f6ff">Word</text>
    <text x="80" y="76" text-anchor="middle" font-family="DM Sans,sans-serif" font-size="22" font-weight="700" fill="#3b82f6">Guess</text>
  </svg>
  <div class="wg-edition">Weekly Puzzle Edition</div>
  <div class="wg-hdate">${dateStr()} &nbsp;&middot;&nbsp; <span>Word Challenge</span></div>
</div>
<div class="wg-controls">
  <button class="wg-btn wg-btn-ghost" onclick="wgHint()">💡 Hint (<span id="wgHC">2</span>)</button>
  <button class="wg-btn wg-btn-ghost" onclick="wgCheck()">✓ Check</button>
  <button class="wg-btn wg-btn-reset" onclick="wgReset()">↺ Reset</button>
</div>
<div class="wg-game">
  <div class="wg-stats-row">
    <div class="wg-sbox"><div class="wg-sbox-lbl">⏱ Time</div><div class="wg-sbox-val t" id="wgTimer">00:00</div></div>
    <div class="wg-sbox"><div class="wg-sbox-lbl">✗ Guesses</div><div class="wg-sbox-val" id="wgGC">0 / 6</div></div>
    <div class="wg-sbox"><div class="wg-sbox-lbl">💡 Hints</div><div class="wg-sbox-val h" id="wgHL">2</div></div>
    <div class="wg-sbox"><div class="wg-sbox-lbl">🔥 Streak</div><div class="wg-sbox-val" id="wgStreak">${stats.streak}</div></div>
  </div>
  <div class="wg-grid" id="wgGrid"></div>
  <div class="wg-hint-card" id="wgHintCard">
    <div class="wg-hint-lbl">Hint — revealed letters</div>
    <div class="wg-hint-tiles" id="wgHintTiles"></div>
  </div>
  <div class="wg-keyboard" id="wgKeyboard"></div>

</div>
<div class="wg-msg" id="wgMsg"></div>
<div class="wg-lb-wrap">
  <div class="wg-lb">
    <div class="wg-lb-head">
      <div class="wg-lb-title">🏆 This Week's Leaderboard</div>
      <div class="wg-lb-sub">Ranked by fastest time</div>
    </div>
    <div id="wgLbList"><div class="wg-lb-loading">Loading scores…</div></div>
    <div class="wg-submit" id="wgSub" style="display:none">
      <div class="wg-submit-lbl">Submit your score</div>
      <div class="wg-submit-row">
        <input class="wg-name-input" id="wgName" type="text" maxlength="20" placeholder="Your name…" autocomplete="off" autocorrect="off" autocapitalize="none">
        <button class="wg-submit-btn" id="wgSubBtn" onclick="wgSubmitScore()">Submit</button>
      </div>
    </div>
  </div>
</div>`;
})();

// ── BUILD GRID ──
function buildGrid(){
  var el=$('wgGrid');if(!el)return;
  el.innerHTML='';
  var sz=tileSz();
  for(var r=0;r<G.maxGuesses;r++){
    var row=document.createElement('div');row.className='wg-row';row.id='wgR'+r;
    for(var c=0;c<G.wordLen;c++){
      var t=document.createElement('div');t.className='wg-tile';t.id='wgT'+r+'_'+c;
      t.style.width=sz+'px';t.style.height=sz+'px';t.style.fontSize=Math.round(sz*0.42)+'px';
      row.appendChild(t);
    }
    el.appendChild(row);
  }
}

// ── BUILD KEYBOARD ──
// ── BUILD KEYBOARD — rebuilds DOM and resets all key colors ──
function buildKB(){
  var kb=$('wgKeyboard');if(!kb)return;
  kb.innerHTML='';
  var ks=keySz();
  [['Q','W','E','R','T','Y','U','I','O','P'],
   ['A','S','D','F','G','H','J','K','L'],
   ['⌫','Z','X','C','V','B','N','M','ENTER']
  ].forEach(function(row){
    var re=document.createElement('div');re.className='wg-kb-row';
    row.forEach(function(k){
      var b=document.createElement('button');
      var wide=(k==='ENTER'||k==='⌫');
      b.className='wg-key'+(wide?' wide':'');
      b.textContent=k;
      b.dataset.key=k;
      b.style.height=ks.h+'px';
      b.style.fontSize=ks.fs+'px';
      b.addEventListener('pointerdown',function(e){e.preventDefault();handleKey(k);});
      re.appendChild(b);
    });
    kb.appendChild(re);
  });
}

// ── RENDER ──
function renderGrid(){
  for(var r=0;r<G.maxGuesses;r++){
    for(var c=0;c<G.wordLen;c++){
      var t=$('wgT'+r+'_'+c);if(!t)continue;
      t.className='wg-tile';
      var sz=tileSz();t.style.width=sz+'px';t.style.height=sz+'px';t.style.fontSize=Math.round(sz*0.42)+'px';
      if(r < G.guesses.length){
        t.textContent = (G.guesses[r][c] || '').toUpperCase();
        // Use saved result — never recalculate after submission
        t.classList.add(G.results[r] ? G.results[r][c] : tileState(G.guesses[r], c));
      }
      else if(r===G.guesses.length&&!G.finished){var l=(G.currentInput[c]||'').toUpperCase();t.textContent=l;if(l)t.classList.add('tbd');}
      else t.textContent='';
    }
  }
  var gc=$('wgGC');if(gc)gc.textContent=G.guesses.length+' / 6';
  var hl=$('wgHL');if(hl)hl.textContent=G.hintsLeft;
  var hc=$('wgHC');if(hc)hc.textContent=G.hintsLeft;
  renderHint();
}

// ── CORE COMPARISON LOGIC ──
// Priority rule: correct (green) first, then present (yellow)
// Duplicate letter rule: each answer letter can only account for ONE result
function guessResult(g) {
  var ans   = G.word.toUpperCase().split('');
  var guess = g.toUpperCase().split('');
  var res   = Array(G.wordLen).fill('absent');
  // answerPool tracks how many of each letter are still "available" for yellow
  var answerPool = {};

  // PASS 1 — lock in all CORRECT (green) positions first
  for (var i = 0; i < G.wordLen; i++) {
    if (guess[i] === ans[i]) {
      res[i] = 'correct';
    } else {
      // only count letters NOT matched green toward the pool
      answerPool[ans[i]] = (answerPool[ans[i]] || 0) + 1;
    }
  }

  // PASS 2 — mark PRESENT (yellow) using remaining pool counts
  for (var i = 0; i < G.wordLen; i++) {
    if (res[i] === 'correct') continue; // already green, skip
    var letter = guess[i];
    if (answerPool[letter] && answerPool[letter] > 0) {
      res[i] = 'present';
      answerPool[letter]--;
    }
    // else stays 'absent'
  }

  return res;
}
function tileState(g, c) { return guessResult(g)[c]; }

function renderHint(){
  var el=$('wgHintTiles');if(!el)return;
  el.innerHTML='';
  for(var i=0;i<G.wordLen;i++){
    var t=document.createElement('div');t.className='wg-htile';
    var rev=G.hintsUsed.indexOf(i)!==-1;
    if(rev){t.textContent=G.word[i];t.classList.add('rev');}else t.textContent='_';
    el.appendChild(t);
  }
}



// ── KEY COLORS: rebuild from G.results every time — no stale state ──
function updateKeys(g, res) {
  // Rebuild entire keyStates from all submitted results (source of truth)
  G.keyStates = {};
  var priority = { correct: 3, present: 2, absent: 1 };

  G.guesses.forEach(function(guess, ri) {
    var result = G.results[ri];
    if (!result) return;
    for (var i = 0; i < guess.length; i++) {
      var letter  = guess[i].toUpperCase();
      var state   = result[i];
      var current = G.keyStates[letter];
      if (!current || priority[state] > priority[current]) {
        G.keyStates[letter] = state;
      }
    }
  });

  // Apply to DOM
  var ks = keySz();
  document.querySelectorAll('.wg-key').forEach(function(b) {
    var k = b.dataset.key;
    b.className = 'wg-key' + ((k === 'ENTER' || k === '⌫') ? ' wide' : '');
    if (G.keyStates[k]) b.classList.add(G.keyStates[k]);
    b.style.height = ks.h + 'px';
    b.style.fontSize = ks.fs + 'px';
  });
}

// ── FLIP ──
// ── FLIP ANIMATION per tile with delay ──
function flipRow(ri, g, res, cb) {
  var FLIP_DELAY   = 300; // ms between each tile flip start
  var FLIP_HALF    = 200; // ms to reach 90° (halfway)
  var FLIP_FULL    = 200; // ms to unfold back to 0°

  for (var c = 0; c < G.wordLen; c++) {
    (function(col) {
      var t = $('wgT' + ri + '_' + col);
      if (!t) return;
      var startDelay = col * FLIP_DELAY;

      setTimeout(function() {
        // First half: fold down (tile turns to edge)
        t.style.transition = 'transform ' + FLIP_HALF + 'ms ease-in';
        t.style.transform  = 'scaleY(0)';

        setTimeout(function() {
          // Mid-point: swap color class, then unfold
          var sz = tileSz();
          t.className   = 'wg-tile ' + res[col];
          t.textContent = g[col].toUpperCase();
          t.style.width = sz + 'px';
          t.style.height = sz + 'px';
          t.style.fontSize = Math.round(sz * 0.42) + 'px';

          t.style.transition = 'transform ' + FLIP_FULL + 'ms ease-out';
          t.style.transform  = 'scaleY(1)';

          setTimeout(function() { t.style.transition = ''; }, FLIP_FULL + 10);
        }, FLIP_HALF);

      }, startDelay);
    })(c);
  }

  // Callback fires after all tiles have finished flipping
  var totalTime = (G.wordLen - 1) * FLIP_DELAY + FLIP_HALF + FLIP_FULL + 60;
  setTimeout(cb, totalTime);
}
function bounceRow(ri){for(var c=0;c<G.wordLen;c++){(function(col){setTimeout(function(){var t=$('wgT'+ri+'_'+col);if(!t)return;t.classList.add('bounce');setTimeout(function(){t.classList.remove('bounce');},750);},col*75);})(c);}}
function shakeRow(ri){var r=$('wgR'+ri);if(!r)return;r.querySelectorAll('.wg-tile').forEach(function(t){t.classList.add('shake');setTimeout(function(){t.classList.remove('shake');},460);});}

// ── TOAST ──
function toast(msg,dur){
  dur=dur||1700;
  var tw=$('wgTW');
  var t=document.createElement('div');t.className='wg-toast';t.textContent=msg;tw.appendChild(t);
  requestAnimationFrame(function(){requestAnimationFrame(function(){t.classList.add('show');});});
  setTimeout(function(){t.classList.remove('show');setTimeout(function(){t.remove();},300);},dur);
}

// ── MSG ──
function showMsg(txt,type){var m=$('wgMsg');m.innerHTML=txt;m.className='wg-msg '+type;}

// ── HANDLE KEY INPUT ──
function handleKey(k) {
  // Block all input during flip animation or after game ends
  if (G.finished || G.isProcessing) return;

  if (k === '⌫' || k === 'Backspace') {
    if (G.currentInput.length > 0) {
      G.currentInput = G.currentInput.slice(0, -1);
      renderGrid();
    }
  } else if (k === 'ENTER' || k === 'Enter') {
    submitGuess();
  } else if (/^[A-Za-z]$/.test(k) && G.currentInput.length < G.wordLen) {
    var letter = k.toUpperCase();
    var col = G.currentInput.length;
    G.currentInput += letter;
    var t = $('wgT' + G.guesses.length + '_' + col);
    if (t) { t.classList.add('pop'); setTimeout(function(){ t.classList.remove('pop'); }, 100); }
    renderGrid();
  }
}
window.wgHandleKey=handleKey;

// ── CONFETTI ──
function launchConfetti(){
  var colors=['#4ade80','#60a5fa','#fbbf24','#f472b6','#a78bfa','#34d399','#38bdf8'];
  for(var i=0;i<80;i++){
    (function(i){
      setTimeout(function(){
        var el=document.createElement('div');
        el.className='wg-confetti';
        el.style.left=Math.random()*100+'vw';
        el.style.background=colors[Math.floor(Math.random()*colors.length)];
        el.style.width=(6+Math.random()*8)+'px';
        el.style.height=(6+Math.random()*8)+'px';
        el.style.borderRadius=Math.random()>.5?'50%':'2px';
        var dur=1.8+Math.random()*1.8;
        el.style.animationDuration=dur+'s';
        el.style.animationDelay='0s';
        document.body.appendChild(el);
        setTimeout(function(){el.remove();},dur*1000+100);
      }, i*30);
    })(i);
  }
}

// ── WIN OVERLAY ──
function showWinOverlay(tries, score){
  var emojis=['🏆','🌟','⚡','🎯','👍','😅'];
  var msgs=['GENIUS!','MAGNIFICENT!','IMPRESSIVE!','SPLENDID!','GREAT!','PHEW!'];

  // Remove any existing overlay
  var old=$('wgWinOverlay');if(old)old.remove();

  var overlay=document.createElement('div');
  overlay.className='wg-win-overlay';
  overlay.id='wgWinOverlay';
  overlay.innerHTML=`
    <div class="wg-win-box">
      <div class="wg-win-emoji">${emojis[tries-1]}</div>
      <div class="wg-win-title">${msgs[tries-1]}</div>
      <div class="wg-win-subtitle">You solved this week's word</div>
      <div class="wg-win-word">${G.word}</div>
      <div class="wg-win-meta">Solved in <span>${tries}</span> ${tries===1?'try':'tries'} &nbsp;·&nbsp; Time: <span>${ft(G.timer)}</span> &nbsp;·&nbsp; Score: <span>${score}</span></div>
      <div class="wg-win-btns">
        <div class="wg-win-submit-row">
          <input class="wg-win-name" id="wgWinName" type="text" maxlength="20" placeholder="Your name for leaderboard…" autocomplete="off" autocorrect="off" autocapitalize="none">
          <button class="wg-win-submit" id="wgWinSubBtn" onclick="wgWinSubmit()">Submit</button>
        </div>
        <button class="wg-win-close" onclick="document.getElementById('wgWinOverlay').remove()">Close &amp; see board</button>
      </div>
    </div>`;
  document.body.appendChild(overlay);

  // Also update the bottom submit area
  var sw=$('wgSub');if(sw)sw.style.display='none';
}

window.wgWinSubmit=function(){
  var nameEl=$('wgWinName');
  var name=(nameEl?nameEl.value:'').trim();
  if(!name){if(nameEl)nameEl.focus();return;}
  var btn=$('wgWinSubBtn');
  if(btn){btn.disabled=true;btn.textContent='Saving…';}
  saveLB(name,function(ok){
    if(ok){
      var ov=$('wgWinOverlay');
      if(ov)ov.remove();
      toast('Score submitted! 🏆',2200);
      setTimeout(loadLB,1400);
    } else {
      if(btn){btn.disabled=false;btn.textContent='Submit';}
      toast('Could not save. Try again.');
    }
  });
};
function submitGuess() {
  var g  = G.currentInput.toUpperCase();
  var ri = G.guesses.length;

  if (g.length < G.wordLen) { shakeRow(ri); toast('Not enough letters'); return; }
  if (!VALID_SET.has(g) && !EXTRA.has(g)) { shakeRow(ri); toast('Not in word list'); return; }

  // Lock input during flip animation
  G.isProcessing = true;

  var res = guessResult(g);
  G.guesses.push(g);
  G.results.push(res);  // save permanently at submission time
  G.currentInput = '';
  renderGrid(); // clear input row immediately

  flipRow(ri, g, res, function() {
    // Animation done — update key colors & re-render
    updateKeys(g, res);
    renderGrid();

    // Unlock input
    G.isProcessing = false;

    var won = res.every(function(r){ return r === 'correct'; });

    if (won) {
      G.finished = true; G.won = true;
      clearInterval(G.timerInt);
      var tr  = G.guesses.length;
      G.score = Math.max(10, (110 - tr * 10) + G.hintsLeft * 4 - Math.floor(G.timer / 30));
      stats.played++; stats.wins++; stats.streak++;
      stats.maxStreak = Math.max(stats.maxStreak, stats.streak);
      stats.dist[tr - 1]++;
      localStorage.setItem('wg3_stats', JSON.stringify(stats));
      var se = $('wgStreak'); if (se) se.textContent = stats.streak;
      // Bounce all tiles in winning row
      bounceRow(ri);
      // Confetti + overlay with delay so bounce plays first
      setTimeout(function(){
        launchConfetti();
        showWinOverlay(tr, G.score);
      }, 700);

    } else if (G.guesses.length >= G.maxGuesses) {
      G.finished = true;
      clearInterval(G.timerInt);
      stats.played++; stats.streak = 0;
      localStorage.setItem('wg3_stats', JSON.stringify(stats));
      var se = $('wgStreak'); if (se) se.textContent = 0;
      setTimeout(function(){
        showMsg('💀 Game Over!  The word was: <strong style="color:#f0f6ff;letter-spacing:.12em">' + G.word + '</strong>', 'over');
      }, 300);
    }

    var gc = $('wgGC'); if (gc) gc.textContent = G.guesses.length + ' / 6';
  });
}

// ── HINT ──
window.wgHint=function(){
  if(G.finished)return;
  if(G.hintsLeft<=0){toast('No hints remaining');return;}
  var un=[];
  for(var i=0;i<G.wordLen;i++){
    var alreadyCorrect=false;
    G.results.forEach(function(r){ if(r[i]==='correct') alreadyCorrect=true; });
    if(!alreadyCorrect && G.hintsUsed.indexOf(i)===-1) un.push(i);
  }
  if(!un.length){toast('All positions already known!');return;}
  var pos=un[Math.floor(Math.random()*un.length)];
  G.hintsUsed.push(pos);G.hintsLeft--;
  $('wgHintCard').style.display='block';
  renderHint();
  var hc=$('wgHC');if(hc)hc.textContent=G.hintsLeft;
  var hl=$('wgHL');if(hl)hl.textContent=G.hintsLeft;
  toast('Position '+(pos+1)+' is "'+G.word[pos]+'"');
};

// ── CHECK ──
window.wgCheck=function(){
  if(G.guesses.length===0){toast('Make a guess first!');return;}
  var res=guessResult(G.guesses[G.guesses.length-1]),cor=0,pre=0;
  res.forEach(function(r){if(r==='correct')cor++;else if(r==='present')pre++;});
  toast(cor+' correct position · '+pre+' wrong position',2200);
};

// ── WEEKLY FIXED WORD (same word for everyone this week) ──
function getWeeklyWord(){
  // Deterministic seed from week key so all players get the same word
  var seed=0;
  for(var i=0;i<WEEK.length;i++) seed=(seed*31+WEEK.charCodeAt(i))>>>0;
  return WORDS[seed % WORDS.length];
}
// ── RESET ──
window.wgReset=function(){
  clearInterval(G.timerInt);
  G.word=getWeeklyWord();
  G.wordLen=5;G.maxGuesses=6;G.hintsLeft=2;G.hintsUsed=[];
  G.guesses=[];G.results=[];G.currentInput='';G.keyStates={};
  G.timer=0;G.finished=false;G.won=false;G.score=0;G.isProcessing=false;
  var m=$('wgMsg');if(m)m.className='wg-msg';
  var sw=$('wgSub');if(sw)sw.style.display='none';
  var hc=$('wgHintCard');if(hc)hc.style.display='none';
  var ov=$('wgWinOverlay');if(ov)ov.remove();
  buildGrid();buildKB();renderGrid();
  var te=$('wgTimer');if(te)te.textContent='00:00';
  G.timerInt=setInterval(function(){if(!G.finished){G.timer++;var te=$('wgTimer');if(te)te.textContent=ft(G.timer);}},1000);
};

// ── LEADERBOARD ──
function fetchLB(cb){
  fetch(SCRIPT_URL+'?week='+encodeURIComponent(WEEK))
    .then(function(r){return r.json();})
    .then(function(d){d.sort(function(a,b){return Number(a.time||99999)-Number(b.time||99999);});cb(d);})
    .catch(function(){cb([]);});
}
function saveLB(name,cb){
  fetch(SCRIPT_URL,{method:'POST',headers:{'Content-Type':'text/plain'},
    body:JSON.stringify({week:WEEK,name:name,tries:G.guesses.length,score:G.score,time:G.timer,hintsLeft:G.hintsLeft,timestamp:Date.now()})})
  .then(function(r){return r.json();}).then(function(d){cb(d.result==='ok');}).catch(function(){cb(false);});
}
function renderLB(scores){
  var el=$('wgLbList');
  if(!scores.length){el.innerHTML='<div class="wg-lb-empty">No scores yet 🎯 Be the first!</div>';return;}
  var medals=['🥇','🥈','🥉'];
  var html='<table class="wg-lb-table"><thead><tr><th>#</th><th>Player</th><th>Time</th><th>Hints Left</th><th>Date</th></tr></thead><tbody>';
  scores.slice(0,10).forEach(function(s,i){
    html+='<tr><td class="wg-rank">'+(medals[i]||'#'+(i+1))+'</td><td class="wg-plr">'+esc(s.name)+'</td><td class="wg-tv">'+ft(Number(s.time||0))+'</td><td class="wg-hv">'+(s.hintsLeft!==undefined?s.hintsLeft:'—')+'</td><td class="wg-dv">'+fd(s.timestamp)+'</td></tr>';
  });
  html+='</tbody></table>';el.innerHTML=html;
}
function loadLB(){fetchLB(renderLB);}

window.wgSubmitScore=function(){
  var ne=$('wgName'),name=(ne.value||'').trim();
  if(!name){ne.focus();return;}
  var btn=$('wgSubBtn');btn.disabled=true;btn.textContent='Saving…';
  saveLB(name,function(ok){
    if(ok){$('wgSub').style.display='none';toast('Score submitted! 🏆',2000);setTimeout(loadLB,1500);}
    else{btn.disabled=false;btn.textContent='Submit';toast('Could not save. Try again.');}
  });
};

// ── KEYBOARD ──
document.addEventListener('keydown', function(e) {
  // Do NOT intercept keyboard when user is typing in any input/textarea
  var tag = document.activeElement && document.activeElement.tagName;
  if (tag === 'INPUT' || tag === 'TEXTAREA') return;
  if (e.ctrlKey || e.metaKey || e.altKey) return;
  if (e.key === 'Enter') { e.preventDefault(); handleKey('Enter'); }
  else if (e.key === 'Backspace') { e.preventDefault(); handleKey('Backspace'); }
  else if (/^[a-zA-Z]$/.test(e.key)) handleKey(e.key.toUpperCase());
});
document.body.addEventListener('touchmove',function(e){
  if(!e.target.closest('.wg-lb,.wg-name-input'))e.preventDefault();
},{passive:false});

// ── START ──
wgReset();
toast('📅 This week\'s word — same for everyone!', 2400);
loadLB();

})();
</script>
</body>
</html>
