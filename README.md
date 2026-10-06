# -
神了
<!doctype html>
<html lang="zh-CN">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>追踪光球</title>
  <style>
    * {
      box-sizing: border-box;
    }

    html,
    body {
      width: 100%;
      height: 100%;
      margin: 0;
      overflow: hidden;
      background: #080b12;
    }

    canvas {
      display: block;
      width: 100vw;
      height: 100vh;
      touch-action: none;
    }

    .overlay {
      position: fixed;
      inset: 0;
      z-index: 10;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 24px;
      background: rgba(2, 5, 10, 0.78);
    }

    .overlay.hidden {
      display: none;
    }

    .panel {
      width: min(720px, 92vw);
      max-height: 92vh;
      padding: 30px;
      border: 2px solid rgba(255, 255, 255, 0.32);
      border-radius: 14px;
      background: #0d1320;
      color: #fff;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.5);
      overflow-y: auto;
    }

    .panel h2 {
      margin: 0 0 18px;
      text-align: center;
      font-size: 28px;
      letter-spacing: 0;
    }

    .panel-subtitle {
      margin: -8px 0 20px;
      text-align: center;
      color: #ffd166;
      font-size: 17px;
    }

    .choices {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 14px;
    }

    .choice,
    .shop-choice {
      position: relative;
      min-height: 150px;
      padding: 18px 14px;
      border: 2px solid rgba(255, 255, 255, 0.22);
      border-radius: 12px;
      background: #121a2a;
      color: #fff;
      font-family: inherit;
      cursor: pointer;
      transition: border-color 0.15s ease, transform 0.15s ease, background 0.15s ease;
    }

    .choice:hover,
    .shop-choice:hover:not(:disabled) {
      border-color: #ffd166;
      background: #172238;
      transform: translateY(-2px);
    }

    .choice-name,
    .shop-choice-name {
      display: block;
      margin-bottom: 12px;
      color: #ffd166;
      font-size: 21px;
      font-weight: 700;
    }

    .choice-desc,
    .shop-choice-desc {
      display: block;
      color: rgba(255, 255, 255, 0.82);
      font-size: 16px;
      line-height: 1.5;
    }

    .shop-choice-price {
      display: block;
      margin-top: 14px;
      color: #ffd166;
      font-size: 16px;
    }

    .shop-lock {
      position: absolute;
      top: 9px;
      right: 9px;
      min-width: 34px;
      height: 34px;
      padding: 0 8px;
      border: 2px solid rgba(255, 255, 255, 0.45);
      border-radius: 8px;
      background: rgba(255, 255, 255, 0.08);
      color: #fff;
      font-size: 14px;
      line-height: 1;
      cursor: pointer;
    }

    .shop-lock.locked {
      border-color: #ffd166;
      background: rgba(255, 209, 102, 0.18);
    }

    .shop-toolbar {
      display: flex;
      align-items: center;
      justify-content: space-between;
      gap: 12px;
      margin: -8px 0 18px;
    }

    .shop-toolbar .panel-subtitle {
      margin: 0;
      text-align: left;
    }

    .shop-refresh {
      padding: 9px 14px;
      border: 2px solid #57b9ff;
      border-radius: 10px;
      background: rgba(87, 185, 255, 0.12);
      color: #dff2ff;
      font-family: inherit;
      font-size: 15px;
      font-weight: 700;
      cursor: pointer;
    }

    .shop-refresh:disabled {
      cursor: not-allowed;
      opacity: 0.45;
    }

    .shop-buy {
      display: block;
      width: 100%;
      margin-top: 14px;
      padding: 9px 12px;
      border: 2px solid #ffd166;
      border-radius: 10px;
      background: #ffd166;
      color: #101827;
      font-family: inherit;
      font-size: 15px;
      font-weight: 800;
      cursor: pointer;
    }

    .shop-buy:disabled {
      cursor: not-allowed;
      opacity: 0.45;
    }

    .shop-choice:disabled {
      cursor: not-allowed;
      opacity: 0.5;
    }

    .next-level {
      display: block;
      width: 100%;
      margin-top: 20px;
      padding: 14px;
      border: 2px solid #ffd166;
      border-radius: 12px;
      background: #ffd166;
      color: #101827;
      font-family: inherit;
      font-size: 18px;
      font-weight: 700;
      cursor: pointer;
    }

    .next-level:hover {
      filter: brightness(1.08);
    }

    @media (max-width: 640px) {
      .choices {
        grid-template-columns: 1fr;
      }

      .choice,
      .shop-choice {
        min-height: 110px;
      }
    }
  </style>
</head>
<body>
  <canvas id="game"></canvas>

  <div id="skillOverlay" class="overlay hidden">
    <div class="panel">
      <h2>升级！选择技能</h2>
      <div id="skillChoices" class="choices"></div>
    </div>
  </div>

  <div id="shopOverlay" class="overlay hidden">
    <div class="panel">
      <h2>武器商店</h2>
      <div class="shop-toolbar">
        <div id="shopCoins" class="panel-subtitle"></div>
        <button id="shopRefreshBtn" class="shop-refresh" type="button">刷新商品</button>
      </div>
      <div id="shopChoices" class="choices"></div>
      <button id="nextLevelBtn" class="next-level" type="button">进入下一关</button>
    </div>
  </div>

  <script>
    (function () {
      const canvas = document.getElementById("game");
      const ctx = canvas.getContext("2d");
      const TAU = Math.PI * 2;
      const skillOverlay = document.getElementById("skillOverlay");
      const skillChoices = document.getElementById("skillChoices");
      const shopOverlay = document.getElementById("shopOverlay");
      const shopChoices = document.getElementById("shopChoices");
      const shopCoins = document.getElementById("shopCoins");
      const shopRefreshBtn = document.getElementById("shopRefreshBtn");
      const nextLevelBtn = document.getElementById("nextLevelBtn");

      let W = 0;
      let H = 0;
      let DPR = 1;

      // ===== 基础参数 =====
      const PLAYER_RADIUS = 25;
      const PLAYER_COLLISION_RADIUS = PLAYER_RADIUS;
      const PLAYER_SPEED = 100;
      const PLAYER_MAX_HP = 100;
      const PLAYER_HURT_COOLDOWN = 0.9;

      // 自动攻击
      const ATTACK_COOLDOWN = 0.8;

      // 光球
      const PROJECTILE_RADIUS = 7;
      const PROJECTILE_SPEED = 500;
      const PROJECTILE_DAMAGE = 15;
      const PROJECTILE_HOMING_STRENGTH = 6.0;
      const PROJECTILE_LIFE = 3.2;
      const PROJECTILE_TRAIL_LENGTH = 16;

      // 怪物
      const MONSTER_RADIUS = 20;
      const MONSTER_COLLISION_RADIUS = MONSTER_RADIUS;
      const MONSTER_SPEED = 70;
      const MONSTER_HP = 40;
      const MONSTER_CONTACT_DAMAGE = 15;
      const SPAWN_INTERVAL = 1.5;
      const MAX_MONSTERS = 20;

      // 掉落与道具
      const LOOT_LIFETIME = 20;
      const AUTO_PICKUP_RANGE = 110;
      const PICKUP_SPEED = 520;
      const PICKUP_DISTANCE = 22;
      const COIN_VALUE = 1;

      const BOMB_DAMAGE = 100;
      const FREEZE_DURATION = 2;
      const MAGNET_DURATION = 5;
      const HEALTH_PACK_HEAL = 50;

      const GRID = 64;
      const LEVEL_TIME_LIMITS = [50, 60, 90, 120];
      const XP_REQUIREMENTS = [5, 10, 15, 20, 30, 50, 60, 70, 90, 100, 130, 150, 180, 200];
      const MAX_LEVEL_XP = 200;

      const keys = {};
      const player = {
        x: 0,
        y: 0,
        hp: PLAYER_MAX_HP,
        facingX: 1,
        facingY: 0,
        attackCooldown: 0,
        lastHurtAt: -10
      };

      let projectiles = [];
      let monsters = [];
      let items = [];
      let effects = [];
      let particles = [];

      let coins = 0;
      let experience = 0;
      let gameTime = 0;
      let spawnTimer = 0;
      let gameOver = false;
      let last = performance.now();
      let gamePaused = false;

      let playerMaxHp = PLAYER_MAX_HP;
      let playerLevel = 1;
      let stage = 1;
      let levelTimeLeft = LEVEL_TIME_LIMITS[0];

      let skillDamageBonus = 0;
      let skillCooldownReduction = 0;
      let skillSpeedBonus = 0;
      let skillExtraProjectiles = 0;
      let pickupRangeBonus = 0;

      let weaponDamageBonus = 0;
      let weaponCooldownReduction = 0;
      let weaponExtraProjectiles = 0;

      // 新商店装备与战斗状态
      const equipment = {
        ribbon: false,
        broom: false,
        reyasKnife: false,
        crossbow: false,
        sword: false,
        pen: false,
        anansBook: false,
        thirteenWater: false,
        noahsSpray: false,
        anansFakeBlood: false,
        firePoker: false
      };

      let spearSynthesized = false;
      let spearCooldown = 0;
      let pierceEffects = [];
      let pathMarks = [];
      let pathMarkTimer = 0;
      let firePatches = [];

      // 希罗的钢笔每次回溯叠加，因此单独记录累计倍率
      let penAttackSpeedMult = 0;
      let penDamageMult = 0;
      let penMaxHpMult = 0;

      // 回溯技能自身的等级成长加成
      let reviveAttackSpeedMult = 0;
      let reviveMoveSpeedMult = 0;
      let reviveDamageMult = 0;
      let reviveSkillCooldownReduction = 0;

      // 技能等级与商店随机货架
      let skillLevels = {
        meteor: 0,
        word: 0,
        revive: 0,
        fireball: 0,
        butterfly: 0,
        look: 0,
        flight: 0,
        force: 0
      };
      let shopSlots = [];
      let shopRefreshUsed = 0;

      let reviveOwned = false;
      let reviveUsedThisLevel = false;

      // 道具持续状态
      let magnetUntil = -10;
      let freezeUntil = -10;

      let activeSkills = [];
      let skillZones = [];
      let wordProjectiles = [];
      let butterflies = [];
      let tauntTexts = [];
      let forceWaves = [];

      let fireballSkill = false;
      let reviveReady = false;
      let flightUnlocked = false;
      let flightActive = false;
      let flightTimeRemaining = 10;
      let flightSession = 0;
      let flightCooldown = 0;
      let flightShadow = null;

      let skillPool = [
        {
          id: "meteor",
          name: "天降安安",
          baseDesc: "天降安安",
          apply() {},
          activate() {
            return castMeteor();
          }
        },
        {
          id: "word",
          name: "言灵",
          baseDesc: "言灵",
          apply() {},
          activate() {
            return castWord();
          }
        },
        {
          id: "revive",
          name: "回溯",
          baseDesc: "回溯",
          apply() {}
        },
        {
          id: "fireball",
          name: "火球",
          baseDesc: "火球",
          apply() {}
        },
        {
          id: "butterfly",
          name: "血蝶",
          baseDesc: "血蝶",
          apply() {},
          activate() {
            return castButterflies();
          }
        },
        {
          id: "look",
          name: "朝这里看",
          baseDesc: "朝这里看",
          apply() {},
          activate() {
            return castLookHere();
          }
        },
        {
          id: "flight",
          name: "飞行",
          baseDesc: "飞行",
          apply() {}
        },
        {
          id: "force",
          name: "怪力",
          baseDesc: "怪力",
          apply() {},
          activate() {
            return castForceWave();
          }
        }
      ];

      const weaponShop = [
        {
          id: "ribbon",
          name: "奈叶香的发带",
          desc: "攻速 +10%",
          cost: 10,
          bought: false,
          apply() {
            equipment.ribbon = true;
          }
        },
        {
          id: "broom",
          name: "扫把",
          desc: "移速 +20%",
          cost: 13,
          bought: false,
          apply() {
            equipment.broom = true;
          }
        },
        {
          id: "reyas-knife",
          name: "蕾雅的刀",
          desc: "攻击力 +40%",
          cost: 20,
          bought: false,
          apply() {
            equipment.reyasKnife = true;
          }
        },
        {
          id: "crossbow",
          name: "弩箭",
          desc: "暴击率 +10%",
          cost: 15,
          bought: false,
          apply() {
            equipment.crossbow = true;
          }
        },
        {
          id: "sword",
          name: "仪礼剑",
          desc: "暴击率 +10%，暴击伤害 +20%",
          cost: 20,
          bought: false,
          apply() {
            equipment.sword = true;
          }
        },
        {
          id: "pen",
          name: "希罗的钢笔",
          desc: "触发回溯复活时，攻速、伤害、最大生命值全部 +50%",
          cost: 30,
          bought: false,
          apply() {
            equipment.pen = true;
          }
        },
        {
          id: "anans-book",
          name: "安安的本子",
          desc: "言灵冷却 -2秒，言灵爆炸范围 +20px",
          cost: 30,
          bought: false,
          condition() {
            return skillPool.some((skill) => skill.id === "word");
          },
          apply() {
            equipment.anansBook = true;
          }
        },
        {
          id: "thirteen-water",
          name: "13水",
          desc: "普通攻击20%概率造成200伤害，同怪仅一次；触发后未死亡则静止2秒并减速20%",
          cost: 30,
          bought: false,
          apply() {
            equipment.thirteenWater = true;
          }
        },
        {
          id: "noahs-spray",
          name: "诺亚的喷漆",
          desc: "普通攻击附带50°角、80px范围白色粒子攻击，额外13伤害，减速15%",
          cost: 25,
          bought: false,
          apply() {
            equipment.noahsSpray = true;
          }
        },
        {
          id: "anans-fake-blood",
          name: "安安的假血",
          desc: "移动路径留下红色痕迹，怪物接触每秒受10伤害，20秒后逐步消失",
          cost: 15,
          bought: false,
          apply() {
            equipment.anansFakeBlood = true;
          }
        },
        {
          id: "fire-poker",
          name: "拔火棍",
          desc: "攻速 +20%，攻击血量低于50%的怪物时额外造成30伤害",
          cost: 20,
          bought: false,
          apply() {
            equipment.firePoker = true;
          }
        }
      ];

      function getSkillLevel(id) {
        return skillLevels[id] || 0;
      }

      function getSkill(skillId) {
        return skillPool.find((skill) => skill.id === skillId);
      }

      function getBaseSkillCooldown(skillId, level) {
        const currentLevel = level || getSkillLevel(skillId);
        if (skillId === "meteor") return currentLevel >= 3 ? 8 : currentLevel >= 2 ? 10 : 15;
        if (skillId === "word") return currentLevel >= 3 ? 5 : currentLevel >= 2 ? 7 : 8;
        if (skillId === "butterfly") return currentLevel >= 2 ? 7 : 9;
        if (skillId === "look") return currentLevel >= 2 ? 8 : 9;
        if (skillId === "force") return currentLevel >= 3 ? 6 : currentLevel >= 2 ? 8 : 10;
        return 0;
      }

      function getSkillCooldown(skillId, level) {
        const currentLevel = level || getSkillLevel(skillId);
        let cooldown = getBaseSkillCooldown(skillId, currentLevel);
        if (skillId === "word" && equipment.anansBook) {
          cooldown = Math.max(1, cooldown - 2);
        }
        if (reviveSkillCooldownReduction > 0) {
          cooldown *= 1 - reviveSkillCooldownReduction;
        }
        return Math.max(1, cooldown);
      }

      function getSkillDescription(skill, level) {
        const targetLevel = level || getSkillLevel(skill.id) + 1;
        const lvl = Math.min(3, Math.max(1, targetLevel));
        if (skill.id === "meteor") {
          if (lvl === 1) return "1级：冷却15秒，范围300px，闪烁1.5秒后生成实心圆，触碰怪物造成50伤害，0.2秒消失。";
          if (lvl === 2) return "2级：冷却10秒，范围350px，闪烁1.2秒，伤害50。";
          return "3级：冷却8秒，范围450px，伤害60；圆生成0.1秒后变红，额外15伤害并将怪物击退100px。";
        }
        if (skill.id === "word") {
          if (lvl === 1) return "1级：冷却8秒，生成“出去”追踪最近怪物；100px内怪物1秒远离玩家80px。";
          if (lvl === 2) return "2级：冷却7秒，范围150px，2秒内远离玩家120px。";
          return "3级：冷却5秒，范围200px，2.5秒内远离玩家180px。";
        }
        if (skill.id === "revive") {
          if (lvl === 1) return "1级：每关生效一次，本关原地复活。";
          if (lvl === 2) return "2级：复活后攻速、移速+20%，伤害+30%，技能冷却-10%，本局生效。";
          return "3级：复活后攻速、移速+40%，伤害+40%，技能冷却-15%。";
        }
        if (skill.id === "fireball") {
          if (lvl === 1) return "1级：火球50px爆炸，20伤害，附加3秒10/秒燃烧。";
          if (lvl === 2) return "2级：爆炸80px，生成0.5秒火堆，接触8/秒伤害。";
          return "3级：爆炸100px，30伤害，击退20px；燃烧20/秒，火堆30/秒。";
        }
        if (skill.id === "butterfly") {
          if (lvl === 1) return "1级：冷却9秒，3只血蝶追踪第2、3、4远怪物，命中后环绕，每0.4秒5伤害，持续2秒并减速20%。";
          if (lvl === 2) return "2级：冷却7秒，4只血蝶；第4只追踪视野血量最高怪物，每秒造成10%最大生命值伤害，累计100后消失。";
          return "3级：4只血蝶；小血蝶追踪第2、3、4近怪物，大血蝶追踪最高血量怪物直至死亡。";
        }
        if (skill.id === "look") {
          if (lvl === 1) return "1级：冷却9秒，全部怪物停止0.5秒，显示“朝这里看”1秒。";
          if (lvl === 2) return "2级：冷却8秒，停止0.8秒，随后怪物减速10%持续5秒。";
          return "3级：冷却8秒，停止1.5秒，随后怪物减速30%持续3秒；持有简易长矛时额外造成10%最大生命值伤害。";
        }
        if (skill.id === "flight") {
          if (lvl === 1) return "1级：每飞行1秒增加4秒冷却；最长10秒；飞行时普攻攻速、移速各+30%。";
          if (lvl === 2) return "2级：每飞行1秒增加3秒冷却；飞行时普攻攻速、移速各+50%。";
          return "3级：每飞行1秒增加1.5秒冷却；飞行时普攻攻速、伤害+50%，移速+60%。";
        }
        if (skill.id === "force") {
          if (lvl === 1) return "1级：冷却10秒，50px触发，60px范围40伤害，1.2秒击飞，落地额外30伤害。";
          if (lvl === 2) return "2级：冷却8秒，80px触发，50伤害，1.5秒击飞，落地30伤害。";
          return "3级：冷却6秒，150px触发，80伤害，1.8秒击飞，落地40伤害，并向外移动100px。";
        }
        return skill.baseDesc;
      }

      function getFlightMoveSpeedMult() {
        if (!flightActive) return 1;
        const level = getSkillLevel("flight");
        return level >= 3 ? 1.6 : level >= 2 ? 1.5 : 1.3;
      }

      function getFlightAttackSpeedMult() {
        if (!flightActive) return 1;
        const level = getSkillLevel("flight");
        return level >= 2 ? 1.5 : 1.3;
      }

      function getFlightDamageMult() {
        return flightActive && getSkillLevel("flight") >= 3 ? 0.5 : 0;
      }

      function currentPlayerSpeed() {
        let speed = PLAYER_SPEED + skillSpeedBonus;
        if (equipment.broom) {
          speed *= 1 + (spearSynthesized ? 0.4 : 0.2);
        }
        speed *= 1 + reviveMoveSpeedMult;
        speed *= getFlightMoveSpeedMult();
        return speed;
      }

      function currentAttackCooldown() {
        let attackSpeedFactor = 1;
        if (equipment.ribbon) {
          attackSpeedFactor += spearSynthesized ? 0.2 : 0.1;
        }
        if (equipment.firePoker) {
          attackSpeedFactor += 0.2;
        }
        attackSpeedFactor += penAttackSpeedMult;
        attackSpeedFactor += reviveAttackSpeedMult;
        attackSpeedFactor += getFlightAttackSpeedMult() - 1;

        return Math.max(
          0.15,
          (ATTACK_COOLDOWN - weaponCooldownReduction - skillCooldownReduction) / attackSpeedFactor
        );
      }

      function currentDamageMultiplier() {
        let damageMultiplier = 1;
        if (equipment.reyasKnife) {
          damageMultiplier += spearSynthesized ? 0.8 : 0.4;
        }
        damageMultiplier += penDamageMult;
        damageMultiplier += reviveDamageMult;
        damageMultiplier += getFlightDamageMult();
        return damageMultiplier;
      }

      function currentProjectileDamage() {
        return (PROJECTILE_DAMAGE + weaponDamageBonus + skillDamageBonus) * currentDamageMultiplier();
      }

      function currentProjectileCount() {
        return 1 + weaponExtraProjectiles + skillExtraProjectiles;
      }

      function currentCritChance() {
        let chance = 0;
        if (equipment.crossbow) {
          chance += spearSynthesized ? 0.2 : 0.1;
        }
        if (equipment.sword) {
          chance += 0.1;
        }
        return chance;
      }

      function currentCritDamage() {
        return 2 + (equipment.sword ? 0.2 : 0);
      }

      function currentWordEffectRadius() {
        const level = getSkillLevel("word");
        let radius = level >= 3 ? 200 : level >= 2 ? 150 : 100;
        if (equipment.anansBook) {
          radius += 20;
        }
        return radius;
      }

      function currentPickupRange() {
        return AUTO_PICKUP_RANGE + pickupRangeBonus;
      }

      function currentLevelTimeLimit() {
        return LEVEL_TIME_LIMITS[Math.min(stage - 1, LEVEL_TIME_LIMITS.length - 1)];
      }

      function currentSpawnInterval() {
        return Math.max(0.8, SPAWN_INTERVAL * Math.pow(0.8, stage - 1));
      }

      function currentMonsterMaxHp() {
        return Math.min(100, Math.round(MONSTER_HP * Math.pow(1.3, stage - 1)));
      }

      function xpRequired(level) {
        if (level <= XP_REQUIREMENTS.length) {
          return XP_REQUIREMENTS[level - 1];
        }
        return MAX_LEVEL_XP;
      }

      function formatTime(seconds) {
        const total = Math.max(0, Math.floor(seconds));
        const minutes = Math.floor(total / 60);
        const rest = total % 60;
        return String(minutes).padStart(2, "0") + ":" + String(rest).padStart(2, "0");
      }

      function pickRandomSkills(count) {
        const pool = skillPool.filter((skill) => getSkillLevel(skill.id) < 3);
        for (let i = pool.length - 1; i > 0; i -= 1) {
          const j = Math.floor(Math.random() * (i + 1));
          const temp = pool[i];
          pool[i] = pool[j];
          pool[j] = temp;
        }
        return pool.slice(0, count);
      }

      function applySkill(skill) {
        const skillId = skill.id;
        skillLevels[skillId] = Math.min(3, getSkillLevel(skillId) + 1);

        if (skillId === "revive") {
          reviveOwned = true;
          if (!reviveUsedThisLevel) {
            reviveReady = true;
          }
        } else if (skillId === "fireball") {
          fireballSkill = true;
        } else if (skillId === "flight") {
          flightUnlocked = true;
        }

        if (["meteor", "word", "butterfly", "look", "force"].includes(skillId)) {
          const alreadyActive = activeSkills.some((entry) => entry.skill.id === skillId);
          if (!alreadyActive) {
            activeSkills.push({
              skill,
              lastUsedAt: -Infinity
            });
          }
        }
      }

      function openSkillPanel() {
        const choices = pickRandomSkills(3);
        if (!choices.length) {
          playerLevel += 1;
          gamePaused = false;
          return;
        }

        skillChoices.innerHTML = "";

        choices.forEach((skill) => {
          const button = document.createElement("button");
          button.type = "button";
          button.className = "choice";

          const nextLevel = getSkillLevel(skill.id) + 1;
          const name = document.createElement("span");
          name.className = "choice-name";
          name.textContent = skill.name + " Lv" + nextLevel;

          const desc = document.createElement("span");
          desc.className = "choice-desc";
          desc.textContent = getSkillDescription(skill, nextLevel);

          button.appendChild(name);
          button.appendChild(desc);
          button.addEventListener("click", () => {
            applySkill(skill);
            playerLevel += 1;
            skillOverlay.classList.add("hidden");
            gamePaused = false;
          });
          skillChoices.appendChild(button);
        });

        skillOverlay.classList.remove("hidden");
      }

      function checkSpearSynthesis() {
        if (spearSynthesized) return;

        if (
          equipment.ribbon &&
          equipment.broom &&
          equipment.reyasKnife &&
          equipment.crossbow
        ) {
          spearSynthesized = true;
          ["ribbon", "broom", "reyas-knife", "crossbow"].forEach((id) => {
            const weapon = weaponShop.find((entry) => entry.id === id);
            if (weapon) {
              weapon.consumed = true;
            }
          });
        }
      }

      function getAvailableShopWeapons() {
        return weaponShop.filter((weapon) => {
          if (weapon.bought || weapon.consumed) return false;
          if (weapon.condition && !weapon.condition()) return false;
          return true;
        });
      }

      function rollShopWeapon(excludedIds) {
        const available = getAvailableShopWeapons().filter((weapon) => {
          return !excludedIds.includes(weapon.id);
        });
        if (!available.length) return null;
        return available[Math.floor(Math.random() * available.length)];
      }

      function initializeShopSlots() {
        if (!shopSlots.length) {
          for (let i = 0; i < 3; i += 1) {
            shopSlots.push({
              weapon: null,
              locked: false
            });
          }
        }
      }

      function cleanInvalidShopSlots() {
        for (let i = 0; i < shopSlots.length; i += 1) {
          const slot = shopSlots[i];
          if (
            slot.weapon &&
            (
              slot.weapon.bought ||
              slot.weapon.consumed ||
              (slot.weapon.condition && !slot.weapon.condition())
            )
          ) {
            slot.weapon = null;
            slot.locked = false;
          }
        }
      }

      function fillUnlockedShopSlots() {
        initializeShopSlots();
        cleanInvalidShopSlots();

        const usedIds = shopSlots
          .filter((slot) => slot.locked && slot.weapon)
          .map((slot) => slot.weapon.id);

        for (let i = 0; i < shopSlots.length; i += 1) {
          const slot = shopSlots[i];
          if (slot.locked && slot.weapon) continue;
          slot.locked = false;
          slot.weapon = rollShopWeapon(usedIds);
          if (slot.weapon) {
            usedIds.push(slot.weapon.id);
          }
        }
      }

      function openShop() {
        gamePaused = true;
        shopRefreshUsed = 0;
        fillUnlockedShopSlots();
        renderShop();
        shopOverlay.classList.remove("hidden");
      }

      function refreshShop() {
        if (shopRefreshUsed >= 1) return;
        shopRefreshUsed += 1;
        fillUnlockedShopSlots();
        renderShop();
      }

      function purchaseShopWeapon(slotIndex) {
        const slot = shopSlots[slotIndex];
        if (!slot || !slot.weapon) return;
        const weapon = slot.weapon;
        if (weapon.bought || coins < weapon.cost) return;

        coins -= weapon.cost;
        weapon.bought = true;
        weapon.apply();
        checkSpearSynthesis();

        slot.weapon = null;
        slot.locked = false;
        cleanInvalidShopSlots();
        renderShop();
      }

      function renderShop() {
        shopCoins.textContent = "当前金币：" + coins +
          (spearSynthesized ? " ｜ 已合成：简易长矛" : "");
        shopRefreshBtn.disabled = shopRefreshUsed >= 1;
        shopChoices.innerHTML = "";

        shopSlots.forEach((slot, index) => {
          const weapon = slot.weapon;
          const card = document.createElement("div");
          card.className = "shop-choice";

          if (!weapon) {
            const emptyName = document.createElement("span");
            emptyName.className = "shop-choice-name";
            emptyName.textContent = "空位";

            const emptyDesc = document.createElement("span");
            emptyDesc.className = "shop-choice-desc";
            emptyDesc.textContent = "刷新后可获得新商品。";

            card.appendChild(emptyName);
            card.appendChild(emptyDesc);
            shopChoices.appendChild(card);
            return;
          }

          const lockButton = document.createElement("button");
          lockButton.type = "button";
          lockButton.className = "shop-lock" + (slot.locked ? " locked" : "");
          lockButton.textContent = slot.locked ? "已锁" : "锁定";
          lockButton.addEventListener("click", (event) => {
            event.stopPropagation();
            slot.locked = !slot.locked;
            renderShop();
          });

          const name = document.createElement("span");
          name.className = "shop-choice-name";
          name.textContent = weapon.name;

          const desc = document.createElement("span");
          desc.className = "shop-choice-desc";
          desc.textContent = weapon.desc;

          const price = document.createElement("span");
          price.className = "shop-choice-price";
          price.textContent = "价格：" + weapon.cost + " 金币";

          const buyButton = document.createElement("button");
          buyButton.type = "button";
          buyButton.className = "shop-buy";
          buyButton.textContent = "购买";
          buyButton.disabled = coins < weapon.cost;
          buyButton.addEventListener("click", (event) => {
            event.stopPropagation();
            purchaseShopWeapon(index);
          });

          card.appendChild(lockButton);
          card.appendChild(name);
          card.appendChild(desc);
          card.appendChild(price);
          card.appendChild(buyButton);
          shopChoices.appendChild(card);
        });
      }

      function finishLevel() {
        gamePaused = true;
        monsters = [];
        projectiles = [];
        skillZones = [];
        wordProjectiles = [];
        butterflies = [];
        tauntTexts = [];
        forceWaves = [];
        pierceEffects = [];
        pathMarks = [];
        pathMarkTimer = 0;
        firePatches = [];
        openShop();
      }

      function startNextLevel() {
        stage += 1;
        levelTimeLeft = currentLevelTimeLimit();
        spawnTimer = currentSpawnInterval();
        reviveUsedThisLevel = false;
        reviveReady = reviveOwned;
        monsters = [];
        projectiles = [];
        skillZones = [];
        wordProjectiles = [];
        butterflies = [];
        tauntTexts = [];
        forceWaves = [];
        pierceEffects = [];
        pathMarks = [];
        pathMarkTimer = 0;
        firePatches = [];
        spearCooldown = 0;
        flightShadow = null;
        flightActive = false;
        flightTimeRemaining = 10;
        flightSession = 0;
        flightCooldown = 0;
        shopOverlay.classList.add("hidden");
        gamePaused = false;
      }

      function checkLevelUp() {
        if (gameOver || gamePaused) return;
        if (experience < xpRequired(playerLevel)) return;

        experience = 0;
        gamePaused = true;
        openSkillPanel();
      }

      function resize() {
        DPR = Math.min(window.devicePixelRatio || 1, 2);
        W = window.innerWidth;
        H = window.innerHeight;

        canvas.width = Math.floor(W * DPR);
        canvas.height = Math.floor(H * DPR);
        canvas.style.width = W + "px";
        canvas.style.height = H + "px";
        ctx.setTransform(DPR, 0, 0, DPR, 0, 0);
      }

      function resetGame() {
        keys.w = false;
        keys.a = false;
        keys.s = false;
        keys.d = false;
        keys.space = false;
        gamePaused = false;
        skillOverlay.classList.add("hidden");
        shopOverlay.classList.add("hidden");

        player.x = W / 2;
        player.y = H / 2;
        playerMaxHp = PLAYER_MAX_HP;
        player.hp = playerMaxHp;
        player.facingX = 0;
        player.facingY = 0;
        player.attackCooldown = 0;
        player.lastHurtAt = -10;

        projectiles = [];
        monsters = [];
        items = [];
        effects = [];
        particles = [];
        activeSkills = [];
        skillZones = [];
        wordProjectiles = [];
        butterflies = [];
        tauntTexts = [];
        forceWaves = [];
        pierceEffects = [];
        pathMarks = [];
        pathMarkTimer = 0;
        firePatches = [];
        shopSlots = [];
        shopRefreshUsed = 0;
        coins = 0;
        experience = 0;
        gameTime = 0;
        spawnTimer = currentSpawnInterval();
        gameOver = false;
        playerLevel = 1;
        stage = 1;
        levelTimeLeft = currentLevelTimeLimit();
        magnetUntil = -10;
        freezeUntil = -10;

        skillDamageBonus = 0;
        skillCooldownReduction = 0;
        skillSpeedBonus = 0;
        skillExtraProjectiles = 0;
        pickupRangeBonus = 0;

        weaponDamageBonus = 0;
        weaponCooldownReduction = 0;
        weaponExtraProjectiles = 0;

        equipment.ribbon = false;
        equipment.broom = false;
        equipment.reyasKnife = false;
        equipment.crossbow = false;
        equipment.sword = false;
        equipment.pen = false;
        equipment.anansBook = false;
        equipment.thirteenWater = false;
        equipment.noahsSpray = false;
        equipment.anansFakeBlood = false;
        equipment.firePoker = false;
        spearSynthesized = false;
        spearCooldown = 0;
        penAttackSpeedMult = 0;
        penDamageMult = 0;
        penMaxHpMult = 0;
        reviveAttackSpeedMult = 0;
        reviveMoveSpeedMult = 0;
        reviveDamageMult = 0;
        reviveSkillCooldownReduction = 0;
        reviveOwned = false;
        reviveUsedThisLevel = false;

        Object.keys(skillLevels).forEach((skillId) => {
          skillLevels[skillId] = 0;
        });

        weaponShop.forEach((weapon) => {
          weapon.bought = false;
          weapon.consumed = false;
        });

        fireballSkill = false;
        reviveReady = false;
        flightUnlocked = false;
        flightActive = false;
        flightTimeRemaining = 10;
        flightSession = 0;
        flightCooldown = 0;
        flightShadow = null;

        last = performance.now();

        for (let i = 0; i < 6; i += 1) {
          spawnMonster();
        }

        player.facingX = 1;
        player.facingY = 0;
      }

      function viewBounds() {
        return {
          left: player.x - W / 2,
          right: player.x + W / 2,
          top: player.y - H / 2,
          bottom: player.y + H / 2
        };
      }

      function inView(x, y, margin) {
        const pad = margin || 0;
        const view = viewBounds();
        return (
          x >= view.left - pad &&
          x <= view.right + pad &&
          y >= view.top - pad &&
          y <= view.bottom + pad
        );
      }

      function monstersInView() {
        return monsters.filter((monster) => !monster.dead && inView(monster.x, monster.y, 20));
      }

      function nearestMonster(x, y) {
        let best = null;
        let bestDistance = Infinity;

        for (let i = 0; i < monsters.length; i += 1) {
          const monster = monsters[i];
          if (monster.dead) continue;
          const dx = monster.x - x;
          const dy = monster.y - y;
          const distance = dx * dx + dy * dy;
          if (distance < bestDistance) {
            bestDistance = distance;
            best = monster;
          }
        }

        return best;
      }

      function addHitEffect(x, y) {
        effects.push({
          x,
          y,
          age: 0,
          life: 0.22,
          maxRadius: 34
        });

        for (let i = 0; i < 12; i += 1) {
          const angle = Math.random() * TAU;
          const speed = 80 + Math.random() * 220;
          particles.push({
            x,
            y,
            vx: Math.cos(angle) * speed,
            vy: Math.sin(angle) * speed,
            age: 0,
            life: 0.2 + Math.random() * 0.28,
            radius: 1.5 + Math.random() * 2.5,
            color: "#ff8a3d"
          });
        }
      }

      function addDeathEffect(x, y) {
        effects.push({
          x,
          y,
          age: 0,
          life: 0.38,
          maxRadius: 58
        });

        for (let i = 0; i < 24; i += 1) {
          const angle = Math.random() * TAU;
          const speed = 60 + Math.random() * 260;
          const white = Math.random() > 0.7;
          particles.push({
            x,
            y,
            vx: Math.cos(angle) * speed,
            vy: Math.sin(angle) * speed,
            age: 0,
            life: 0.25 + Math.random() * 0.45,
            radius: 1.2 + Math.random() * 3.4,
            color: white ? "#ffffff" : "#ffb347"
          });
        }
      }

      function addPickupEffect(x, y, color) {
        effects.push({
          x,
          y,
          age: 0,
          life: 0.2,
          maxRadius: 24
        });

        for (let i = 0; i < 7; i += 1) {
          const angle = Math.random() * TAU;
          const speed = 50 + Math.random() * 150;
          particles.push({
            x,
            y,
            vx: Math.cos(angle) * speed,
            vy: Math.sin(angle) * speed,
            age: 0,
            life: 0.2 + Math.random() * 0.2,
            radius: 1.2 + Math.random() * 2.2,
            color
          });
        }
      }

      function addPlayerHurtEffect(x, y) {
        effects.push({
          x,
          y,
          age: 0,
          life: 0.24,
          maxRadius: 44
        });
      }

      function itemRadius(type) {
        if (type === "xp") return 8;
        if (type === "coin") return 9;
        return 13;
      }

      function addItem(type, x, y) {
        const angle = Math.random() * TAU;
        const speed = 20 + Math.random() * 50;
        const permanent = type === "xp" || type === "coin";
        items.push({
          x: x + (Math.random() * 2 - 1) * 8,
          y: y + (Math.random() * 2 - 1) * 8,
          vx: Math.cos(angle) * speed,
          vy: Math.sin(angle) * speed,
          type,
          radius: itemRadius(type),
          value: type === "coin" ? COIN_VALUE : 1,
          life: permanent ? Infinity : LOOT_LIFETIME,
          attract: false
        });
      }

      function dropLoot(monster) {
        addItem("xp", monster.x, monster.y);

        const roll = Math.random();
        if (roll < 0.30) {
          addItem("coin", monster.x, monster.y);
        } else if (roll < 0.31) {
          addItem("bomb", monster.x, monster.y);
        } else if (roll < 0.32) {
          addItem("freeze", monster.x, monster.y);
        } else if (roll < 0.33) {
          addItem("magnet", monster.x, monster.y);
        } else if (roll < 0.34) {
          addItem("health", monster.x, monster.y);
        }
      }

      function spawnMonster() {
        if (monsters.length >= MAX_MONSTERS) return;

        const spawnRadius = Math.hypot(W, H) / 2 + 100 + Math.random() * 180;
        let baseAngle;

        if (player.facingX !== 0 || player.facingY !== 0) {
          baseAngle = Math.atan2(player.facingY, player.facingX);
          baseAngle += (Math.random() - 0.5) * 1.35;
        } else {
          baseAngle = Math.random() * TAU;
        }

        const x = player.x + Math.cos(baseAngle) * spawnRadius;
        const y = player.y + Math.sin(baseAngle) * spawnRadius;

        monsters.push({
          x,
          y,
          vx: 0,
          vy: 0,
          hp: currentMonsterMaxHp(),
          maxHp: currentMonsterMaxHp(),
          radius: MONSTER_RADIUS,
          seed: Math.random() * TAU,
          hitFlash: 0,
          wordAwayUntil: -10,
          wordStopUntil: -10,
          wordAwayX: 0,
          wordAwayY: 0,
          wordAwaySpeed: 50,
          stopUntil: -10,
          burnUntil: -10,
          burnDps: 10,
          slowFactor: 1,
          lookSlowUntil: -10,
          lookSlowFactor: 1,
          knockbackUntil: -10,
          knockbackX: 0,
          knockbackY: 0,
          knockupUntil: -10,
          knockupVx: 0,
          knockupVy: 0,
          knockupDuration: 1.2,
          knockupLandDamage: 0,
          knockupLanded: false,
          thirteenTriggered: false,
          thirteenStopUntil: -10,
          dead: false
        });
      }

      function fireProjectile() {
        if (gameOver || player.attackCooldown > 0) return;

        const target = nearestMonster(player.x, player.y);
        if (!target) return;

        player.attackCooldown = currentAttackCooldown();
        const angle = Math.atan2(target.y - player.y, target.x - player.x);
        const startDistance = PLAYER_COLLISION_RADIUS + 7;
        const count = currentProjectileCount();

        for (let shot = 0; shot < count; shot += 1) {
          const offset = count > 1 ? (shot - (count - 1) / 2) * 0.09 : 0;
          const shotAngle = angle + offset;

          projectiles.push({
            x: player.x + Math.cos(shotAngle) * startDistance,
            y: player.y + Math.sin(shotAngle) * startDistance,
            vx: Math.cos(shotAngle) * PROJECTILE_SPEED,
            vy: Math.sin(shotAngle) * PROJECTILE_SPEED,
            target,
            life: PROJECTILE_LIFE,
            trail: [],
            damage: currentProjectileDamage(),
            type: fireballSkill ? "fireball" : "normal"
          });
        }
      }

      function damageMonster(monster, amount, hitX, hitY) {
        if (monster.dead) return;

        monster.hp -= amount;
        monster.hitFlash = 0.15;
        addHitEffect(hitX, hitY);

        if (monster.hp <= 0 && !monster.dead) {
          killMonster(monster);
        }
      }

      function addCritEffect(x, y) {
        effects.push({
          x,
          y,
          age: 0,
          life: 0.18,
          maxRadius: 42
        });

        for (let i = 0; i < 8; i += 1) {
          const angle = Math.random() * TAU;
          const speed = 90 + Math.random() * 180;
          particles.push({
            x,
            y,
            vx: Math.cos(angle) * speed,
            vy: Math.sin(angle) * speed,
            age: 0,
            life: 0.18 + Math.random() * 0.22,
            radius: 1.2 + Math.random() * 2,
            color: "#ffffff"
          });
        }
      }

      function applySpray(x, y, dirX, dirY) {
        if (!equipment.noahsSpray) return;

        const directionAngle = Math.atan2(dirY, dirX);
        const halfCone = (50 * Math.PI) / 180 / 2;

        for (let i = 0; i < monsters.length; i += 1) {
          const monster = monsters[i];
          if (monster.dead) continue;

          const dx = monster.x - x;
          const dy = monster.y - y;
          if (dx * dx + dy * dy > 6400) continue;

          let angle = Math.atan2(dy, dx) - directionAngle;
          while (angle > Math.PI) angle -= TAU;
          while (angle < -Math.PI) angle += TAU;
          if (Math.abs(angle) > halfCone) continue;

          monster.slowFactor = Math.min(monster.slowFactor, 0.85);
          damageMonster(monster, 13, monster.x, monster.y);
        }

        for (let i = 0; i < 16; i += 1) {
          const angle = directionAngle + (Math.random() - 0.5) * halfCone * 2;
          const speed = 90 + Math.random() * 190;
          particles.push({
            x,
            y,
            vx: Math.cos(angle) * speed,
            vy: Math.sin(angle) * speed,
            age: 0,
            life: 0.18 + Math.random() * 0.24,
            radius: 1 + Math.random() * 2.1,
            color: "#ffffff"
          });
        }
      }

      function damageMonsterWithAttack(monster, baseDamage, hitX, hitY, dirX, dirY) {
        if (monster.dead) return;

        let damage = baseDamage;
        let crit = false;
        let thirteenTriggered = false;

        if (equipment.thirteenWater && !monster.thirteenTriggered) {
          if (Math.random() < 0.2) {
            thirteenTriggered = true;
            monster.thirteenTriggered = true;
            damage = 200;
          }
        }

        if (!thirteenTriggered && Math.random() < currentCritChance()) {
          crit = true;
          damage *= currentCritDamage();
        }

        if (
          !thirteenTriggered &&
          equipment.firePoker &&
          monster.hp < monster.maxHp * 0.5
        ) {
          damage += 30;
        }

        damageMonster(monster, damage, hitX, hitY);

        if (crit) {
          addCritEffect(hitX, hitY);
        }

        if (thirteenTriggered && !monster.dead) {
          monster.thirteenStopUntil = gameTime + 2;
          monster.slowFactor = Math.min(monster.slowFactor, 0.8);
          effects.push({
            x: hitX,
            y: hitY,
            age: 0,
            life: 0.38,
            maxRadius: 64
          });
        }

        applySpray(hitX, hitY, dirX, dirY);
      }

      function updateSpear(dt) {
        if (!spearSynthesized) return;

        spearCooldown -= dt;
        if (spearCooldown > 0) return;

        const target = nearestMonster(player.x, player.y);
        if (!target) return;

        const dx = target.x - player.x;
        const dy = target.y - player.y;
        if (dx * dx + dy * dy > 10000) return;

        damageMonster(target, 20, target.x, target.y);
        spearCooldown = 0.8;
        pierceEffects.push({
          x1: player.x,
          y1: player.y,
          x2: target.x,
          y2: target.y,
          age: 0,
          life: 0.18
        });
      }

      function updatePathMarks(dt) {
        if (!equipment.anansFakeBlood) {
          pathMarks = [];
          pathMarkTimer = 0;
          return;
        }

        const isMoving = keys.w || keys.a || keys.s || keys.d;
        if (isMoving) {
          pathMarkTimer += dt;
          if (pathMarkTimer >= 0.055) {
            pathMarkTimer = 0;
            pathMarks.push({
              x: player.x,
              y: player.y,
              age: 0,
              life: 20
            });
          }
        } else {
          pathMarkTimer = 0;
        }

        for (let i = pathMarks.length - 1; i >= 0; i -= 1) {
          pathMarks[i].age += dt;
          if (pathMarks[i].age >= pathMarks[i].life) {
            pathMarks.splice(i, 1);
          }
        }

        const markRadius = MONSTER_COLLISION_RADIUS + 4;
        const markRadiusSq = markRadius * markRadius;

        for (let i = 0; i < monsters.length; i += 1) {
          const monster = monsters[i];
          if (monster.dead) continue;

          let touching = false;
          for (let j = 0; j < pathMarks.length; j += 1) {
            const mark = pathMarks[j];
            const dx = monster.x - mark.x;
            const dy = monster.y - mark.y;
            if (dx * dx + dy * dy <= markRadiusSq) {
              touching = true;
              break;
            }
          }

          if (!touching) continue;

          monster.hp -= 10 * dt;
          monster.hitFlash = Math.max(monster.hitFlash, 0.05);

          if (monster.hp <= 0 && !monster.dead) {
            killMonster(monster);
          }
        }
      }

      function updateFirePatches(dt) {
        for (let i = firePatches.length - 1; i >= 0; i -= 1) {
          const patch = firePatches[i];
          patch.age += dt;
          if (patch.age >= patch.life) {
            firePatches.splice(i, 1);
            continue;
          }

          const radiusSq = patch.radius * patch.radius;
          for (let j = 0; j < monsters.length; j += 1) {
            const monster = monsters[j];
            if (monster.dead) continue;
            const dx = monster.x - patch.x;
            const dy = monster.y - patch.y;
            if (dx * dx + dy * dy <= radiusSq) {
              monster.hp -= patch.dps * dt;
              monster.hitFlash = Math.max(monster.hitFlash, 0.05);
              if (monster.hp <= 0 && !monster.dead) {
                killMonster(monster);
              }
            }
          }
        }
      }

      function killMonster(monster) {
        if (monster.dead) return;
        monster.dead = true;
        dropLoot(monster);
        addDeathEffect(monster.x, monster.y);
      }

      function activateBomb() {
        const visible = monstersInView();
        for (let i = 0; i < visible.length; i += 1) {
          damageMonster(visible[i], BOMB_DAMAGE, visible[i].x, visible[i].y);
        }

        effects.push({
          x: player.x,
          y: player.y,
          age: 0,
          life: 0.5,
          maxRadius: 110
        });
      }

      function collectItem(item) {
        if (item.type === "xp") {
          experience += item.value;
          addPickupEffect(item.x, item.y, "#6ff7ff");
        } else if (item.type === "coin") {
          coins += item.value;
          addPickupEffect(item.x, item.y, "#ffd166");
        } else if (item.type === "bomb") {
          activateBomb();
          addPickupEffect(item.x, item.y, "#ff7043");
        } else if (item.type === "freeze") {
          freezeUntil = gameTime + FREEZE_DURATION;
          addPickupEffect(item.x, item.y, "#7fd8ff");
        } else if (item.type === "magnet") {
          magnetUntil = gameTime + MAGNET_DURATION;
          addPickupEffect(item.x, item.y, "#ffca3a");
        } else if (item.type === "health") {
          player.hp = Math.min(playerMaxHp, player.hp + HEALTH_PACK_HEAL);
          addPickupEffect(item.x, item.y, "#ff5b7a");
        }
      }

      function distanceToPlayerSq(monster) {
        const dx = monster.x - player.x;
        const dy = monster.y - player.y;
        return dx * dx + dy * dy;
      }

      function castMeteor() {
        const target = nearestMonster(player.x, player.y);
        if (!target) return false;

        const level = getSkillLevel("meteor");
        const range = level >= 3 ? 450 : level >= 2 ? 350 : 300;
        const flash = level >= 2 ? 1.2 : 1.5;
        const radius = range;

        skillZones.push({
          x: target.x,
          y: target.y,
          age: 0,
          life: flash + 0.2,
          damageAt: flash,
          damaged: false,
          secondaryAt: level >= 3 ? flash + 0.1 : Infinity,
          secondaryDone: false,
          radius,
          level
        });
        return true;
      }

      function castWord() {
        const target = nearestMonster(player.x, player.y);
        if (!target) return false;

        wordProjectiles.push({
          x: player.x,
          y: player.y,
          vx: 0,
          vy: 0,
          target,
          text: "出去",
          speed: 340,
          life: 3.2
        });
        return true;
      }

      function castButterflies() {
        const alive = monsters.filter((monster) => !monster.dead);
        if (!alive.length) return false;

        const level = getSkillLevel("butterfly");
        const byDistance = alive.slice().sort((a, b) => {
          return distanceToPlayerSq(a) - distanceToPlayerSq(b);
        });
        const byFarthest = alive.slice().sort((a, b) => {
          return distanceToPlayerSq(b) - distanceToPlayerSq(a);
        });
        const smallTargets = level >= 3
          ? byDistance.slice(1, 4)
          : byFarthest.slice(1, 4);

        const smallCount = level >= 2 ? 3 : 3;
        const smallDps = level >= 3 ? 20 : level >= 2 ? 10 : 5;
        const orbitLife = level >= 3 ? Infinity : level >= 2 ? 2.4 : 2;

        for (let index = 0; index < smallCount; index += 1) {
          const target = smallTargets[index];
          if (!target) continue;

          const angle = Math.atan2(target.y - player.y, target.x - player.x);
          const startX = player.x + Math.cos(angle + (index - 1) * 0.25) * 24;
          const startY = player.y + Math.sin(angle + (index - 1) * 0.25) * 24;

          butterflies.push({
            x: startX,
            y: startY,
            target,
            speed: 125,
            phase: index * 1.8,
            state: "fly",
            type: "small",
            tickTimer: 0,
            tickInterval: 0.4,
            damagePerTick: smallDps,
            orbitLife,
            totalDamage: 0,
            level,
            mineReady: level >= 3
          });
        }

        if (level >= 2) {
          const bigCandidates = level === 2
            ? alive.filter((monster) => inView(monster.x, monster.y, 20))
            : alive;
          const bigTarget = (bigCandidates.length ? bigCandidates : alive).reduce((best, monster) => {
            if (!best || monster.maxHp > best.maxHp) return monster;
            return best;
          }, null);

          const angle = Math.atan2(bigTarget.y - player.y, bigTarget.x - player.x);
          butterflies.push({
            x: player.x + Math.cos(angle) * 26,
            y: player.y + Math.sin(angle) * 26,
            target: bigTarget,
            speed: 105,
            phase: 3,
            state: "fly",
            type: "big",
            tickTimer: 0,
            tickInterval: 1,
            damagePerTick: 0,
            orbitLife: Infinity,
            totalDamage: 0,
            level
          });
        }

        return true;
      }

      function castLookHere() {
        if (!monsters.some((monster) => !monster.dead)) return false;

        const level = getSkillLevel("look");
        const stopDuration = level >= 3 ? 1.5 : level >= 2 ? 0.8 : 0.5;

        for (let i = 0; i < monsters.length; i += 1) {
          const monster = monsters[i];
          if (monster.dead) continue;

          monster.stopUntil = Math.max(monster.stopUntil, gameTime + stopDuration);

          if (level >= 2) {
            const slowDuration = level >= 3 ? 3 : 5;
            monster.lookSlowUntil = gameTime + stopDuration + slowDuration;
            monster.lookSlowFactor = level >= 3 ? 0.7 : 0.9;
          }

          if (level >= 3 && spearSynthesized) {
            damageMonster(monster, monster.maxHp * 0.1, monster.x, monster.y);
          }
        }

        tauntTexts.push({
          x: player.x,
          y: player.y,
          age: 0,
          life: 1,
          text: "朝这里看"
        });
        return true;
      }

      function hasNearbyMonsterWithin(radius) {
        const radiusSq = radius * radius;
        for (let i = 0; i < monsters.length; i += 1) {
          if (!monsters[i].dead && distanceToPlayerSq(monsters[i]) <= radiusSq) {
            return true;
          }
        }
        return false;
      }

      function castForceWave() {
        const level = getSkillLevel("force");
        const triggerRadius = level >= 3 ? 150 : level >= 2 ? 80 : 50;
        const effectRadius = level >= 3 ? 150 : level >= 2 ? 80 : 60;
        const damage = level >= 3 ? 80 : level >= 2 ? 50 : 40;
        const landDamage = level >= 3 ? 40 : 30;
        const duration = level >= 3 ? 1.8 : level >= 2 ? 1.5 : 1.2;

        if (!hasNearbyMonsterWithin(triggerRadius)) return false;

        for (let i = 0; i < monsters.length; i += 1) {
          const monster = monsters[i];
          if (monster.dead) continue;

          const dx = monster.x - player.x;
          const dy = monster.y - player.y;
          const distance = Math.hypot(dx, dy);
          if (distance > effectRadius) continue;

          const pushX = distance > 0 ? dx / distance : Math.random() * 2 - 1;
          const pushY = distance > 0 ? dy / distance : Math.random() * 2 - 1;

          damageMonster(monster, damage, monster.x, monster.y);

          const awayDistance = level >= 3 ? 100 : 0;
          monster.knockupUntil = gameTime + duration;
          monster.knockupDuration = duration;
          monster.knockupVx = pushX * (awayDistance / duration);
          monster.knockupVy = pushY * (awayDistance / duration);
          monster.knockupLandDamage = landDamage;
          monster.knockupLanded = false;
        }

        forceWaves.push({
          x: player.x,
          y: player.y,
          age: 0,
          life: 0.45,
          maxRadius: effectRadius
        });
        return true;
      }

      function applyWordEffect(x, y) {
        const radius = currentWordEffectRadius();
        const radiusSq = radius * radius;
        const level = getSkillLevel("word");
        const duration = level >= 3 ? 2.5 : level >= 2 ? 2 : 1;
        const moveDistance = level >= 3 ? 180 : level >= 2 ? 120 : 80;

        for (let i = 0; i < monsters.length; i += 1) {
          const monster = monsters[i];
          if (monster.dead) continue;

          const dx = monster.x - x;
          const dy = monster.y - y;
          if (dx * dx + dy * dy > radiusSq) continue;

          const awayX = monster.x - player.x;
          const awayY = monster.y - player.y;
          const awayDistance = Math.hypot(awayX, awayY);

          monster.wordAwayUntil = gameTime + duration;
          monster.wordStopUntil = -10;
          monster.wordAwayX = awayDistance > 0 ? awayX / awayDistance : 1;
          monster.wordAwayY = awayDistance > 0 ? awayY / awayDistance : 0;
          monster.wordAwaySpeed = moveDistance / duration;
        }

        for (let i = 0; i < 18; i += 1) {
          const angle = Math.random() * TAU;
          const speed = 60 + Math.random() * 190;
          particles.push({
            x,
            y,
            vx: Math.cos(angle) * speed,
            vy: Math.sin(angle) * speed,
            age: 0,
            life: 0.28 + Math.random() * 0.35,
            radius: 1.4 + Math.random() * 2.8,
            color: "#ff2d55"
          });
        }
      }

      function applyBurn(monster, duration, dps) {
        monster.burnUntil = gameTime + duration;
        monster.burnDps = dps;
      }

      function handlePlayerDeath() {
        if (reviveReady) {
          reviveReady = false;
          reviveUsedThisLevel = true;

          const reviveLevel = getSkillLevel("revive");
          if (reviveLevel >= 2) {
            reviveAttackSpeedMult = reviveLevel >= 3 ? 0.4 : 0.2;
            reviveMoveSpeedMult = reviveLevel >= 3 ? 0.4 : 0.2;
            reviveDamageMult = reviveLevel >= 3 ? 0.4 : 0.3;
            reviveSkillCooldownReduction = reviveLevel >= 3 ? 0.15 : 0.1;
          }

          if (equipment.pen) {
            penAttackSpeedMult += 0.5;
            penDamageMult += 0.5;
            penMaxHpMult += 0.5;
            playerMaxHp = Math.round(PLAYER_MAX_HP * (1 + penMaxHpMult));
          }

          player.hp = playerMaxHp;
          player.lastHurtAt = gameTime;
          player.attackCooldown = 0;
          gameOver = false;
          gamePaused = false;
          tauntTexts.push({
            x: player.x,
            y: player.y - 48,
            age: 0,
            life: 2,
            text: "这是...回溯了吗"
          });
          return;
        }

        player.hp = 0;
        gameOver = true;
        addDeathEffect(player.x, player.y);
      }

      function restartCurrentLevel() {
        player.hp = playerMaxHp;
        player.x = W / 2;
        player.y = H / 2;
        player.lastHurtAt = -10;
        player.attackCooldown = 0;

        monsters = [];
        projectiles = [];
        items = items.filter((item) => item.type === "xp" || item.type === "coin");
        effects = [];
        particles = [];
        skillZones = [];
        wordProjectiles = [];
        butterflies = [];
        tauntTexts = [];
        forceWaves = [];
        pierceEffects = [];
        pathMarks = [];
        pathMarkTimer = 0;
        firePatches = [];
        spearCooldown = 0;

        flightShadow = null;
        flightActive = false;
        flightTimeRemaining = 10;
        flightSession = 0;
        flightCooldown = 0;

        magnetUntil = -10;
        freezeUntil = -10;
        spawnTimer = currentSpawnInterval();
        levelTimeLeft = currentLevelTimeLimit();
        gameOver = false;
        gamePaused = false;
      }

      function updateFlight(dt) {
        if (!flightUnlocked) return;

        const level = getSkillLevel("flight");
        const cooldownPerSecond = (level >= 3 ? 1.5 : level >= 2 ? 3 : 4) *
          (1 - reviveSkillCooldownReduction);

        if (flightActive) {
          flightSession += dt;
          flightTimeRemaining -= dt;

          if (!keys.space || flightTimeRemaining <= 0) {
            flightCooldown = Math.ceil(flightSession) * cooldownPerSecond;
            flightActive = false;
            flightSession = 0;
            flightShadow = null;
            flightTimeRemaining = 10;
          }
        } else {
          flightCooldown = Math.max(0, flightCooldown - dt);

          if (keys.space && flightCooldown <= 0) {
            flightActive = true;
            flightSession = 0;
            flightShadow = {
              x: player.x,
              y: player.y
            };
          }
        }
      }

      function updateActiveSkills(dt) {
        for (let i = 0; i < activeSkills.length; i += 1) {
          const entry = activeSkills[i];
          const cooldown = getSkillCooldown(entry.skill.id);
          if (gameTime - entry.lastUsedAt < cooldown) continue;

          if (entry.skill.activate()) {
            entry.lastUsedAt = gameTime;
          }
        }
      }

      function updateSkillEffects(dt) {
        for (let i = skillZones.length - 1; i >= 0; i -= 1) {
          const zone = skillZones[i];
          zone.age += dt;

          if (!zone.damaged && zone.age >= zone.damageAt) {
            zone.damaged = true;
            const radiusSq = zone.radius * zone.radius;
            const primaryDamage = zone.level >= 3 ? 60 : 50;
            for (let j = 0; j < monsters.length; j += 1) {
              const monster = monsters[j];
              if (monster.dead) continue;
              const dx = monster.x - zone.x;
              const dy = monster.y - zone.y;
              if (dx * dx + dy * dy <= radiusSq) {
                damageMonster(monster, primaryDamage, zone.x, zone.y);
              }
            }
          }

          if (!zone.secondaryDone && zone.level >= 3 && zone.age >= zone.secondaryAt) {
            zone.secondaryDone = true;
            const radiusSq = zone.radius * zone.radius;
            for (let j = 0; j < monsters.length; j += 1) {
              const monster = monsters[j];
              if (monster.dead) continue;
              const dx = monster.x - zone.x;
              const dy = monster.y - zone.y;
              if (dx * dx + dy * dy <= radiusSq) {
                damageMonster(monster, 15, monster.x, monster.y);

                const awayX = monster.x - player.x;
                const awayY = monster.y - player.y;
                const awayDistance = Math.hypot(awayX, awayY) || 1;
                const knockDistance = 100;
                const knockDuration = 0.18;
                monster.knockbackUntil = gameTime + knockDuration;
                monster.knockbackX = (awayX / awayDistance) * (knockDistance / knockDuration);
                monster.knockbackY = (awayY / awayDistance) * (knockDistance / knockDuration);
              }
            }
          }

          if (zone.age >= zone.life) {
            skillZones.splice(i, 1);
          }
        }

        for (let i = wordProjectiles.length - 1; i >= 0; i -= 1) {
          const word = wordProjectiles[i];
          word.life -= dt;

          if (word.life <= 0 || !word.target || word.target.dead) {
            wordProjectiles.splice(i, 1);
            continue;
          }

          const angle = Math.atan2(word.target.y - word.y, word.target.x - word.x);
          word.x += Math.cos(angle) * word.speed * dt;
          word.y += Math.sin(angle) * word.speed * dt;

          let hit = false;
          for (let j = 0; j < monsters.length; j += 1) {
            const monster = monsters[j];
            if (monster.dead) continue;
            const dx = word.x - monster.x;
            const dy = word.y - monster.y;
            if (dx * dx + dy * dy <= (monster.radius + 8) * (monster.radius + 8)) {
              applyWordEffect(word.x, word.y);
              hit = true;
              break;
            }
          }

          if (hit) {
            wordProjectiles.splice(i, 1);
          }
        }

        for (let i = butterflies.length - 1; i >= 0; i -= 1) {
          const butterfly = butterflies[i];

          if (butterfly.type === "big" && (!butterfly.target || butterfly.target.dead)) {
            butterfly.target = monsters.reduce((best, monster) => {
              if (monster.dead) return best;
              if (!best || monster.maxHp > best.maxHp) return monster;
              return best;
            }, null);

            if (!butterfly.target) {
              butterflies.splice(i, 1);
              continue;
            }

            butterfly.state = "fly";
          }

          if (butterfly.state === "fly") {
            if (!butterfly.target || butterfly.target.dead) {
              if (butterfly.type === "small" && butterfly.level >= 3) {
                butterfly.target = null;
                butterfly.state = "mine";
                continue;
              }

              butterflies.splice(i, 1);
              continue;
            }

            const angle = Math.atan2(
              butterfly.target.y - butterfly.y,
              butterfly.target.x - butterfly.x
            );
            butterfly.x += Math.cos(angle) * butterfly.speed * dt;
            butterfly.y += Math.sin(angle) * butterfly.speed * dt;

            const dx = butterfly.x - butterfly.target.x;
            const dy = butterfly.y - butterfly.target.y;
            const reachDistance = butterfly.target.radius + (butterfly.type === "big" ? 16 : 10);
            if (dx * dx + dy * dy <= reachDistance * reachDistance) {
              butterfly.state = "orbit";
              butterfly.tickTimer = 0;

              if (butterfly.type === "small" && butterfly.level === 1) {
                butterfly.target.slowFactor = Math.min(butterfly.target.slowFactor, 0.8);
              }
            }

            continue;
          }

          if (butterfly.state === "mine") {
            let triggeredMonster = null;
            for (let j = 0; j < monsters.length; j += 1) {
              const monster = monsters[j];
              if (monster.dead) continue;
              const dx = monster.x - butterfly.x;
              const dy = monster.y - butterfly.y;
              if (dx * dx + dy * dy <= 10000) {
                triggeredMonster = monster;
                break;
              }
            }

            if (triggeredMonster) {
              const burstDamage = triggeredMonster.maxHp * 0.2;
              for (let j = 0; j < monsters.length; j += 1) {
                const monster = monsters[j];
                if (monster.dead) continue;
                const dx = monster.x - butterfly.x;
                const dy = monster.y - butterfly.y;
                if (dx * dx + dy * dy <= 10000) {
                  damageMonster(monster, burstDamage, butterfly.x, butterfly.y);
                }
              }
              butterflies.splice(i, 1);
            }
            continue;
          }

          if (butterfly.state === "orbit") {
            if (!butterfly.target || butterfly.target.dead) {
              if (butterfly.type === "small" && butterfly.level >= 3) {
                butterfly.target = null;
                butterfly.state = "mine";
              } else {
                butterflies.splice(i, 1);
              }
              continue;
            }

            butterfly.orbitLife -= dt;
            if (butterfly.type === "small" && butterfly.orbitLife <= 0) {
              butterflies.splice(i, 1);
              continue;
            }

            const orbitAngle = gameTime * 4 + butterfly.phase;
            const orbitRadius = butterfly.target.radius + (butterfly.type === "big" ? 14 : 9);
            butterfly.x = butterfly.target.x + Math.cos(orbitAngle) * orbitRadius;
            butterfly.y = butterfly.target.y + Math.sin(orbitAngle) * orbitRadius;

            butterfly.tickTimer -= dt;
            if (butterfly.tickTimer <= 0) {
              butterfly.tickTimer = butterfly.tickInterval;

              const tickDamage = butterfly.type === "big"
                ? butterfly.target.maxHp * 0.1
                : butterfly.damagePerTick;

              damageMonster(butterfly.target, tickDamage, butterfly.x, butterfly.y);
              butterfly.totalDamage += tickDamage;

              if (butterfly.type === "big" && butterfly.level === 2 && butterfly.totalDamage > 100) {
                butterflies.splice(i, 1);
              }
            }
          }
        }

        for (let i = tauntTexts.length - 1; i >= 0; i -= 1) {
          tauntTexts[i].age += dt;
          if (tauntTexts[i].age >= tauntTexts[i].life) {
            tauntTexts.splice(i, 1);
          }
        }

        for (let i = forceWaves.length - 1; i >= 0; i -= 1) {
          forceWaves[i].age += dt;
          if (forceWaves[i].age >= forceWaves[i].life) {
            forceWaves.splice(i, 1);
          }
        }
      }

      function update(dt) {
        gameTime += dt;
        levelTimeLeft -= dt;

        if (levelTimeLeft <= 0) {
          levelTimeLeft = 0;
          finishLevel();
          return;
        }

        updateFlight(dt);
        updateActiveSkills(dt);
        player.attackCooldown = Math.max(0, player.attackCooldown - dt);

        // WASD 移动
        let moveX = (keys.d ? 1 : 0) - (keys.a ? 1 : 0);
        let moveY = (keys.s ? 1 : 0) - (keys.w ? 1 : 0);

        if (moveX !== 0 || moveY !== 0) {
          const length = Math.hypot(moveX, moveY);
          moveX /= length;
          moveY /= length;
          player.facingX = moveX;
          player.facingY = moveY;
          player.x += moveX * currentPlayerSpeed() * dt;
          player.y += moveY * currentPlayerSpeed() * dt;
        }

        if (flightActive && flightShadow) {
          flightShadow.x = player.x;
          flightShadow.y = player.y;
        }

        // 自动攻击：无需按键
        fireProjectile();
        updateSpear(dt);
        updatePathMarks(dt);
        updateFirePatches(dt);

        // 在视野外持续刷怪
        spawnTimer -= dt;
        if (spawnTimer <= 0) {
          spawnMonster();
          spawnTimer = currentSpawnInterval();
        }

        // 怪物追踪玩家
        for (let i = 0; i < monsters.length; i += 1) {
          const monster = monsters[i];
          if (monster.dead) continue;
          monster.hitFlash = Math.max(0, monster.hitFlash - dt);

          if (gameTime < monster.burnUntil) {
            monster.hp -= (monster.burnDps || 10) * dt;

            if (Math.random() < dt * 14) {
              particles.push({
                x: monster.x + (Math.random() * 2 - 1) * 12,
                y: monster.y + (Math.random() * 2 - 1) * 12,
                vx: (Math.random() * 2 - 1) * 30,
                vy: -50 - Math.random() * 40,
                age: 0,
                life: 0.25 + Math.random() * 0.3,
                radius: 1.4 + Math.random() * 2.2,
                color: "#ff7a2f"
              });
            }

            if (monster.hp <= 0 && !monster.dead) {
              killMonster(monster);
              continue;
            }
          }

          if (
            monster.knockupUntil > -10 &&
            gameTime >= monster.knockupUntil &&
            !monster.knockupLanded
          ) {
            monster.knockupLanded = true;
            if (monster.knockupLandDamage > 0) {
              damageMonster(monster, monster.knockupLandDamage, monster.x, monster.y);
            }
            monster.knockupUntil = -10;
            if (monster.dead) {
              continue;
            }
          }

          const targetX = flightActive && flightShadow ? flightShadow.x : player.x;
          const targetY = flightActive && flightShadow ? flightShadow.y : player.y;

          if (gameTime < monster.knockbackUntil) {
            monster.vx = monster.knockbackX;
            monster.vy = monster.knockbackY;
            monster.x += monster.vx * dt;
            monster.y += monster.vy * dt;
          } else if (gameTime < monster.knockupUntil) {
            monster.vx = monster.knockupVx;
            monster.vy = monster.knockupVy;
            monster.x += monster.vx * dt;
            monster.y += monster.vy * dt;
          } else if (gameTime < monster.wordAwayUntil) {
            monster.vx = monster.wordAwayX * (monster.wordAwaySpeed || 50);
            monster.vy = monster.wordAwayY * (monster.wordAwaySpeed || 50);
            monster.x += monster.vx * dt;
            monster.y += monster.vy * dt;
          } else if (
            gameTime < monster.wordStopUntil ||
            gameTime < monster.stopUntil ||
            gameTime < monster.thirteenStopUntil ||
            gameTime < freezeUntil
          ) {
            monster.vx = 0;
            monster.vy = 0;
          } else {
            const angle = Math.atan2(targetY - monster.y, targetX - monster.x);
            let speed = MONSTER_SPEED * (monster.slowFactor || 1);
            if (gameTime < monster.lookSlowUntil) {
              speed *= monster.lookSlowFactor;
            }
            monster.vx = Math.cos(angle) * speed;
            monster.vy = Math.sin(angle) * speed;
            monster.x += monster.vx * dt;
            monster.y += monster.vy * dt;
          }

          // 怪物触碰玩家扣血
          if (
            flightActive ||
            gameTime < monster.knockupUntil ||
            gameTime < monster.knockbackUntil
          ) {
            continue;
          }

          const dx = player.x - monster.x;
          const dy = player.y - monster.y;
          const distance = Math.hypot(dx, dy);
          const collisionDistance = PLAYER_COLLISION_RADIUS + MONSTER_COLLISION_RADIUS;

          if (distance < collisionDistance) {
            const overlap = collisionDistance - distance;
            let pushX;
            let pushY;

            if (distance > 0) {
              pushX = dx / distance;
              pushY = dy / distance;
            } else {
              const pushAngle = Math.random() * TAU;
              pushX = Math.cos(pushAngle);
              pushY = Math.sin(pushAngle);
            }

            monster.x -= pushX * overlap * 0.5;
            monster.y -= pushY * overlap * 0.5;
            player.x += pushX * overlap * 0.5;
            player.y += pushY * overlap * 0.5;

            if (gameTime - player.lastHurtAt > PLAYER_HURT_COOLDOWN) {
              player.hp -= MONSTER_CONTACT_DAMAGE;
              player.lastHurtAt = gameTime;
              addPlayerHurtEffect(player.x, player.y);

              if (player.hp <= 0) {
                handlePlayerDeath();
                return;
              }
            }
          }
        }

        // 怪物之间也不能重合
        for (let i = 0; i < monsters.length; i += 1) {
          const monsterA = monsters[i];
          if (monsterA.dead) continue;

          for (let j = i + 1; j < monsters.length; j += 1) {
            const monsterB = monsters[j];
            if (monsterB.dead) continue;

            const dx = monsterB.x - monsterA.x;
            const dy = monsterB.y - monsterA.y;
            const distance = Math.hypot(dx, dy);
            const minDistance = MONSTER_COLLISION_RADIUS * 2;

            if (distance >= minDistance) continue;

            const overlap = minDistance - distance;
            let pushX;
            let pushY;

            if (distance > 0) {
              pushX = dx / distance;
              pushY = dy / distance;
            } else {
              const pushAngle = Math.random() * TAU;
              pushX = Math.cos(pushAngle);
              pushY = Math.sin(pushAngle);
            }

            const halfOverlap = overlap * 0.5;
            monsterA.x -= pushX * halfOverlap;
            monsterA.y -= pushY * halfOverlap;
            monsterB.x += pushX * halfOverlap;
            monsterB.y += pushY * halfOverlap;
          }
        }

        // 更新追踪光球
        for (let i = projectiles.length - 1; i >= 0; i -= 1) {
          const projectile = projectiles[i];
          projectile.life -= dt;

          if (projectile.life <= 0) {
            projectiles.splice(i, 1);
            continue;
          }

          if (projectile.target && projectile.target.dead) {
            projectile.target = null;
          }

          if (projectile.target) {
            const desiredAngle = Math.atan2(
              projectile.target.y - projectile.y,
              projectile.target.x - projectile.x
            );
            let currentAngle = Math.atan2(projectile.vy, projectile.vx);
            let turn = desiredAngle - currentAngle;

            while (turn > Math.PI) turn -= TAU;
            while (turn < -Math.PI) turn += TAU;

            const maxTurn = PROJECTILE_HOMING_STRENGTH * dt;
            currentAngle += Math.max(-maxTurn, Math.min(maxTurn, turn));

            const speed = Math.hypot(projectile.vx, projectile.vy) || PROJECTILE_SPEED;
            projectile.vx = Math.cos(currentAngle) * speed;
            projectile.vy = Math.sin(currentAngle) * speed;
          }

          projectile.x += projectile.vx * dt;
          projectile.y += projectile.vy * dt;

          projectile.trail.push({ x: projectile.x, y: projectile.y });
          if (projectile.trail.length > PROJECTILE_TRAIL_LENGTH) {
            projectile.trail.shift();
          }

          const viewMargin = 160;
          if (!inView(projectile.x, projectile.y, viewMargin)) {
            projectiles.splice(i, 1);
            continue;
          }

          let hit = false;
          if (projectile.type === "fireball") {
            const fireLevel = getSkillLevel("fireball");
            const explosionRadius = fireLevel >= 3 ? 100 : fireLevel >= 2 ? 80 : 50;
            const explosionDamage = (fireLevel >= 3 ? 30 : 20) * currentDamageMultiplier();
            const burnDps = fireLevel >= 3 ? 20 : 10;
            const radiusSq = explosionRadius * explosionRadius;

            for (let j = 0; j < monsters.length; j += 1) {
              const monster = monsters[j];
              if (monster.dead) continue;

              const dx = projectile.x - monster.x;
              const dy = projectile.y - monster.y;
              const hitDistance = PROJECTILE_RADIUS + MONSTER_COLLISION_RADIUS;

              if (dx * dx + dy * dy <= hitDistance * hitDistance) {
                for (let k = 0; k < monsters.length; k += 1) {
                  const nearby = monsters[k];
                  if (nearby.dead) continue;
                  const ndx = nearby.x - projectile.x;
                  const ndy = nearby.y - projectile.y;
                  if (ndx * ndx + ndy * ndy <= radiusSq) {
                    damageMonster(nearby, explosionDamage, projectile.x, projectile.y);
                    applyBurn(nearby, 3, burnDps);

                    if (fireLevel >= 3 && !nearby.dead) {
                      const awayX = nearby.x - projectile.x;
                      const awayY = nearby.y - projectile.y;
                      const awayDistance = Math.hypot(awayX, awayY) || 1;
                      const knockDuration = 0.18;
                      nearby.knockbackUntil = gameTime + knockDuration;
                      nearby.knockbackX = (awayX / awayDistance) * (20 / knockDuration);
                      nearby.knockbackY = (awayY / awayDistance) * (20 / knockDuration);
                    }
                  }
                }

                if (fireLevel >= 2) {
                  firePatches.push({
                    x: projectile.x,
                    y: projectile.y,
                    age: 0,
                    life: 0.5,
                    dps: fireLevel >= 3 ? 30 : 8,
                    radius: 22
                  });
                }

                effects.push({
                  x: projectile.x,
                  y: projectile.y,
                  age: 0,
                  life: 0.38,
                  maxRadius: explosionRadius
                });
                hit = true;
                break;
              }
            }
          } else {
            for (let j = 0; j < monsters.length; j += 1) {
              const monster = monsters[j];
              if (monster.dead) continue;

              const dx = projectile.x - monster.x;
              const dy = projectile.y - monster.y;
              const hitDistance = PROJECTILE_RADIUS + MONSTER_COLLISION_RADIUS;

              if (dx * dx + dy * dy <= hitDistance * hitDistance) {
                damageMonsterWithAttack(
                  monster,
                  projectile.damage,
                  projectile.x,
                  projectile.y,
                  projectile.vx,
                  projectile.vy
                );
                hit = true;
                break;
              }
            }
          }

          if (hit) {
            projectiles.splice(i, 1);
          }
        }

        // 更新掉落物
        for (let i = items.length - 1; i >= 0; i -= 1) {
          const item = items[i];

          if (Number.isFinite(item.life)) {
            item.life -= dt;

            if (item.life <= 0) {
              items.splice(i, 1);
              continue;
            }
          }

          const dx = player.x - item.x;
          const dy = player.y - item.y;
          const distance = Math.hypot(dx, dy);

          if (item.type === "xp" || item.type === "coin") {
            if (gameTime < magnetUntil && inView(item.x, item.y, 30)) {
              item.attract = true;
            } else if (distance < currentPickupRange()) {
              item.attract = true;
            }
          }

          if (item.attract) {
            const angle = Math.atan2(dy, dx);
            item.vx = Math.cos(angle) * PICKUP_SPEED;
            item.vy = Math.sin(angle) * PICKUP_SPEED;
            item.x += item.vx * dt;
            item.y += item.vy * dt;
          } else {
            item.x += item.vx * dt;
            item.y += item.vy * dt;
            item.vx *= 0.94;
            item.vy *= 0.94;
          }

          const currentDistance = Math.hypot(player.x - item.x, player.y - item.y);
          if (currentDistance < PLAYER_COLLISION_RADIUS + item.radius + 4) {
            collectItem(item);
            items.splice(i, 1);
          }
        }

        monsters = monsters.filter((monster) => {
          if (monster.dead) return false;
          const dx = monster.x - player.x;
          const dy = monster.y - player.y;
          const cullDistance = Math.hypot(W, H) / 2 + 900;
          return dx * dx + dy * dy <= cullDistance * cullDistance;
        });

        for (let i = pierceEffects.length - 1; i >= 0; i -= 1) {
          pierceEffects[i].age += dt;
          if (pierceEffects[i].age >= pierceEffects[i].life) {
            pierceEffects.splice(i, 1);
          }
        }

        // 更新特效与粒子
        for (let i = effects.length - 1; i >= 0; i -= 1) {
          effects[i].age += dt;
          if (effects[i].age >= effects[i].life) {
            effects.splice(i, 1);
          }
        }

        for (let i = particles.length - 1; i >= 0; i -= 1) {
          const particle = particles[i];
          particle.age += dt;
          particle.x += particle.vx * dt;
          particle.y += particle.vy * dt;
          particle.vx *= 0.94;
          particle.vy *= 0.94;

          if (particle.age >= particle.life) {
            particles.splice(i, 1);
          }
        }

        updateSkillEffects(dt);
        checkLevelUp();
      }

      function drawBackground() {
        ctx.fillStyle = "#080b12";
        ctx.fillRect(0, 0, W, H);

        ctx.save();
        ctx.translate(W / 2 - player.x, H / 2 - player.y);

        const viewLeft = player.x - W / 2;
        const viewRight = player.x + W / 2;
        const viewTop = player.y - H / 2;
        const viewBottom = player.y + H / 2;
        const startX = Math.floor(viewLeft / GRID) * GRID;
        const startY = Math.floor(viewTop / GRID) * GRID;

        ctx.strokeStyle = "rgba(120, 160, 210, 0.07)";
        ctx.lineWidth = 1;
        ctx.beginPath();
        for (let x = startX; x <= viewRight; x += GRID) {
          ctx.moveTo(x, viewTop);
          ctx.lineTo(x, viewBottom);
        }
        for (let y = startY; y <= viewBottom; y += GRID) {
          ctx.moveTo(viewLeft, y);
          ctx.lineTo(viewRight, y);
        }
        ctx.stroke();
        ctx.restore();
      }

      function drawPlayer() {
        const invulnerable = gameTime - player.lastHurtAt < PLAYER_HURT_COOLDOWN;
        const blink = invulnerable && Math.floor(gameTime * 20) % 2 === 0;

        ctx.save();
        ctx.translate(player.x, player.y - (flightActive ? 14 : 0));
        ctx.globalAlpha = blink ? 0.4 : 1;

        if (gameTime < magnetUntil) {
          const pulse = 34 + Math.sin(gameTime * 8) * 3;
          ctx.strokeStyle = "rgba(255, 202, 58, 0.65)";
          ctx.lineWidth = 2;
          ctx.beginPath();
          ctx.arc(0, 0, pulse, 0, TAU);
          ctx.stroke();
        }

        const glow = ctx.createRadialGradient(0, 0, 2, 0, 0, PLAYER_RADIUS + 16);
        glow.addColorStop(0, "rgba(96, 229, 255, 0.9)");
        glow.addColorStop(0.25, "rgba(45, 184, 255, 0.5)");
        glow.addColorStop(1, "rgba(45, 184, 255, 0)");
        ctx.fillStyle = glow;
        ctx.beginPath();
        ctx.arc(0, 0, PLAYER_RADIUS + 16, 0, TAU);
        ctx.fill();

        const body = ctx.createRadialGradient(-4, -5, 1, 0, 0, PLAYER_RADIUS);
        body.addColorStop(0, "#ffffff");
        body.addColorStop(0.35, "#9ff3ff");
        body.addColorStop(1, "#1987c9");
        ctx.fillStyle = body;
        ctx.beginPath();
        ctx.arc(0, 0, PLAYER_RADIUS, 0, TAU);
        ctx.fill();

        ctx.strokeStyle = "rgba(230, 252, 255, 0.9)";
        ctx.lineWidth = 2;
        ctx.stroke();

        const angle = Math.atan2(player.facingY, player.facingX);
        ctx.rotate(angle);
        ctx.fillStyle = "#ffffff";
        ctx.beginPath();
        ctx.moveTo(PLAYER_RADIUS + 6, 0);
        ctx.lineTo(PLAYER_RADIUS - 4, -5);
        ctx.lineTo(PLAYER_RADIUS - 4, 5);
        ctx.closePath();
        ctx.fill();

        ctx.restore();
      }

      function drawMonster(monster) {
        const frozen = gameTime < freezeUntil;
        const burning = gameTime < monster.burnUntil;
        const knockedUp = gameTime < monster.knockupUntil;
        const lift = knockedUp
          ? Math.sin((1 - (monster.knockupUntil - gameTime) / (monster.knockupDuration || 1.2)) * Math.PI) * 16
          : 0;
        const pulse = frozen ? 1 : 1 + Math.sin(gameTime * 8 + monster.seed) * 0.06;
        const radius = monster.radius * pulse;

        ctx.save();
        ctx.translate(monster.x, monster.y - lift);
        ctx.shadowColor = burning
          ? "rgba(255, 130, 45, 0.75)"
          : frozen
            ? "rgba(95, 210, 255, 0.6)"
            : "rgba(255, 65, 90, 0.5)";
        ctx.shadowBlur = 14;

        const body = ctx.createRadialGradient(
          -radius * 0.3,
          -radius * 0.35,
          1,
          0,
          0,
          radius
        );

        if (frozen) {
          body.addColorStop(0, "#9eeaff");
          body.addColorStop(0.55, "#4da9d6");
          body.addColorStop(1, "#163c54");
        } else if (burning) {
          body.addColorStop(0, "#ffb347");
          body.addColorStop(0.55, "#e05a25");
          body.addColorStop(1, "#51190f");
        } else {
          body.addColorStop(0, "#e24c68");
          body.addColorStop(0.55, "#a51d42");
          body.addColorStop(1, "#451024");
        }

        ctx.fillStyle = body;
        ctx.beginPath();
        ctx.arc(0, 0, radius, 0, TAU);
        ctx.fill();

        ctx.shadowBlur = 0;
        ctx.strokeStyle = frozen
          ? "rgba(190, 242, 255, 0.8)"
          : "rgba(255, 160, 180, 0.55)";
        ctx.lineWidth = 2;
        ctx.stroke();

        if (monster.hitFlash > 0) {
          ctx.fillStyle = "rgba(255, 240, 230, 0.8)";
          ctx.beginPath();
          ctx.arc(0, 0, radius, 0, TAU);
          ctx.fill();
        }

        if (monster.hp < monster.maxHp) {
          const barWidth = radius * 1.8;
          const barHeight = 5;
          const x = -barWidth / 2;
          const y = -radius - 12;
          const ratio = Math.max(0, monster.hp / monster.maxHp);

          ctx.fillStyle = "rgba(0, 0, 0, 0.48)";
          ctx.fillRect(x - 2, y - 2, barWidth + 4, barHeight + 4);
          ctx.fillStyle = frozen ? "#9eeaff" : "#ff5b7a";
          ctx.fillRect(x, y, barWidth * ratio, barHeight);
        }

        ctx.restore();
      }

      function drawItem(item) {
        const pulse = 1 + Math.sin(gameTime * 7 + item.x * 0.01) * 0.08;
        const radius = item.radius * pulse;

        ctx.save();
        ctx.translate(item.x, item.y);
        ctx.shadowBlur = 12;

        if (item.type === "xp") {
          ctx.shadowColor = "rgba(111, 247, 255, 0.75)";
          ctx.fillStyle = "#5deeff";
          ctx.beginPath();
          ctx.moveTo(0, -radius);
          ctx.lineTo(radius * 0.72, 0);
          ctx.lineTo(0, radius);
          ctx.lineTo(-radius * 0.72, 0);
          ctx.closePath();
          ctx.fill();
          ctx.fillStyle = "#ffffff";
          ctx.beginPath();
          ctx.arc(0, 0, 2, 0, TAU);
          ctx.fill();
        } else if (item.type === "coin") {
          ctx.shadowColor = "rgba(255, 209, 102, 0.7)";
          ctx.fillStyle = "#ffd166";
          ctx.beginPath();
          ctx.arc(0, 0, radius, 0, TAU);
          ctx.fill();
          ctx.fillStyle = "#f4a62a";
          ctx.beginPath();
          ctx.arc(0, 0, radius * 0.52, 0, TAU);
          ctx.fill();
        } else if (item.type === "bomb") {
          ctx.shadowColor = "rgba(255, 90, 60, 0.65)";
          ctx.fillStyle = "#20242c";
          ctx.beginPath();
          ctx.arc(0, 0, radius, 0, TAU);
          ctx.fill();
          ctx.fillStyle = "#ff7043";
          ctx.beginPath();
          ctx.arc(3, -5, 2.5, 0, TAU);
          ctx.fill();
        } else if (item.type === "freeze") {
          ctx.shadowColor = "rgba(127, 216, 255, 0.75)";
          ctx.strokeStyle = "#aeeaff";
          ctx.lineWidth = 3;
          ctx.beginPath();
          for (let i = 0; i < 3; i += 1) {
            const angle = -Math.PI / 2 + (i * TAU) / 3;
            ctx.moveTo(0, 0);
            ctx.lineTo(Math.cos(angle) * radius * 0.85, Math.sin(angle) * radius * 0.85);
          }
          ctx.stroke();
        } else if (item.type === "magnet") {
          ctx.shadowColor = "rgba(255, 202, 58, 0.7)";
          ctx.strokeStyle = "#ffca3a";
          ctx.lineWidth = 5;
          ctx.beginPath();
          ctx.arc(0, 1, radius * 0.58, Math.PI * 0.15, Math.PI * 0.85);
          ctx.stroke();
          ctx.fillStyle = "#ffffff";
          ctx.fillRect(-radius * 0.68, -3, 4, 7);
          ctx.fillRect(radius * 0.55, -3, 4, 7);
        } else if (item.type === "health") {
          ctx.shadowColor = "rgba(255, 91, 122, 0.75)";
          ctx.fillStyle = "#ffffff";
          ctx.beginPath();
          ctx.arc(0, 0, radius, 0, TAU);
          ctx.fill();
          ctx.fillStyle = "#ff5b7a";
          ctx.fillRect(-radius * 0.62, -radius * 0.18, radius * 1.24, radius * 0.36);
          ctx.fillRect(-radius * 0.18, -radius * 0.62, radius * 0.36, radius * 1.24);
        }

        ctx.restore();
      }

      function drawProjectile(projectile) {
        const isFireball = projectile.type === "fireball";
        const coreRadius = isFireball ? PROJECTILE_RADIUS * 1.5 : PROJECTILE_RADIUS;

        for (let i = 0; i < projectile.trail.length; i += 1) {
          const point = projectile.trail[i];
          const ratio = i / projectile.trail.length;
          ctx.fillStyle = isFireball
            ? "rgba(255, 70, 25, " + (0.05 + ratio * 0.24) + ")"
            : "rgba(255, 110, 45, " + (0.04 + ratio * 0.18) + ")";
          ctx.beginPath();
          ctx.arc(point.x, point.y, coreRadius * (0.35 + ratio * 0.55), 0, TAU);
          ctx.fill();
        }

        ctx.save();
        ctx.globalCompositeOperation = "lighter";
        const glow = ctx.createRadialGradient(
          projectile.x,
          projectile.y,
          0,
          projectile.x,
          projectile.y,
          coreRadius * (isFireball ? 3.2 : 2.5)
        );
        glow.addColorStop(0, "rgba(255, 255, 255, 0.95)");
        glow.addColorStop(0.25, isFireball ? "rgba(255, 180, 60, 0.85)" : "rgba(255, 210, 130, 0.75)");
        glow.addColorStop(0.6, isFireball ? "rgba(255, 80, 20, 0.35)" : "rgba(255, 100, 40, 0.25)");
        glow.addColorStop(1, isFireball ? "rgba(255, 45, 10, 0)" : "rgba(255, 80, 30, 0)");
        ctx.fillStyle = glow;
        ctx.beginPath();
        ctx.arc(projectile.x, projectile.y, coreRadius * (isFireball ? 3.2 : 2.5), 0, TAU);
        ctx.fill();
        ctx.restore();

        ctx.fillStyle = "#ffffff";
        ctx.beginPath();
        ctx.arc(projectile.x, projectile.y, coreRadius * 0.42, 0, TAU);
        ctx.fill();
      }

      function drawPathMarks() {
        for (let i = 0; i < pathMarks.length; i += 1) {
          const mark = pathMarks[i];
          const alpha = Math.max(0, 1 - mark.age / mark.life);
          ctx.fillStyle = "rgba(255, 35, 65, " + (alpha * 0.42) + ")";
          ctx.beginPath();
          ctx.arc(mark.x, mark.y, 5, 0, TAU);
          ctx.fill();
        }
      }

      function drawPierceEffects() {
        for (let i = 0; i < pierceEffects.length; i += 1) {
          const attack = pierceEffects[i];
          const progress = attack.age / attack.life;
          const alpha = 1 - progress;

          ctx.save();
          ctx.globalAlpha = alpha;
          ctx.strokeStyle = "#fff7cf";
          ctx.lineWidth = 4;
          ctx.shadowColor = "rgba(255, 238, 170, 0.9)";
          ctx.shadowBlur = 14;
          ctx.beginPath();
          ctx.moveTo(attack.x1, attack.y1);
          ctx.lineTo(attack.x2, attack.y2);
          ctx.stroke();
          ctx.restore();
        }
      }

      function drawFirePatches() {
        for (let i = 0; i < firePatches.length; i += 1) {
          const patch = firePatches[i];
          const alpha = Math.max(0, 1 - patch.age / patch.life);
          const radius = patch.radius * (0.8 + (1 - alpha) * 0.2);

          ctx.save();
          ctx.globalAlpha = alpha * 0.85;
          ctx.fillStyle = "#ff5a1f";
          ctx.shadowColor = "rgba(255, 85, 25, 0.8)";
          ctx.shadowBlur = 16;
          ctx.beginPath();
          ctx.arc(patch.x, patch.y, radius, 0, TAU);
          ctx.fill();
          ctx.fillStyle = "#ffc36b";
          ctx.beginPath();
          ctx.arc(patch.x, patch.y, radius * 0.5, 0, TAU);
          ctx.fill();
          ctx.restore();
        }
      }

      function drawSkillEffects() {
        if (flightActive && flightShadow) {
          ctx.fillStyle = "rgba(0, 0, 0, 0.6)";
          ctx.beginPath();
          ctx.arc(flightShadow.x, flightShadow.y, PLAYER_COLLISION_RADIUS, 0, TAU);
          ctx.fill();
        }

        for (let i = 0; i < skillZones.length; i += 1) {
          const zone = skillZones[i];
          const progress = zone.age / zone.life;
          const alpha = Math.max(0, 1 - progress);
          const radius = zone.radius;
          const redPhase = zone.level >= 3 && zone.age >= zone.secondaryAt;

          ctx.save();
          ctx.globalAlpha = alpha;
          if (redPhase) {
            ctx.fillStyle = "rgba(255, 70, 65, 0.48)";
          } else if (zone.age < zone.damageAt) {
            ctx.fillStyle = Math.floor(zone.age * 20) % 2 === 0
              ? "rgba(255, 240, 180, 0.28)"
              : "rgba(255, 210, 100, 0.12)";
          } else {
            ctx.fillStyle = "rgba(255, 220, 90, 0.42)";
          }
          ctx.beginPath();
          ctx.arc(zone.x, zone.y, radius, 0, TAU);
          ctx.fill();

          ctx.strokeStyle = redPhase ? "rgba(255, 105, 95, 0.95)" : "rgba(255, 244, 200, 0.8)";
          ctx.lineWidth = 3;
          ctx.stroke();
          ctx.restore();
        }

        for (let i = 0; i < wordProjectiles.length; i += 1) {
          const word = wordProjectiles[i];
          ctx.save();
          ctx.font = "700 28px -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif";
          ctx.textAlign = "center";
          ctx.textBaseline = "middle";
          ctx.fillStyle = "#ff2d55";
          ctx.shadowColor = "rgba(255, 45, 85, 0.8)";
          ctx.shadowBlur = 12;
          ctx.fillText(word.text, word.x, word.y);
          ctx.restore();
        }

        for (let i = 0; i < butterflies.length; i += 1) {
          const butterfly = butterflies[i];
          const wing = Math.sin(gameTime * 18 + butterfly.phase) * 4;
          const scale = butterfly.type === "big" ? 1.6 : 1;

          ctx.save();
          ctx.translate(butterfly.x, butterfly.y);
          ctx.rotate(butterfly.target
            ? Math.atan2(butterfly.target.y - butterfly.y, butterfly.target.x - butterfly.x)
            : 0);
          ctx.scale(scale, scale);
          ctx.fillStyle = "#ff3b5c";
          ctx.shadowColor = "rgba(255, 59, 92, 0.7)";
          ctx.shadowBlur = 10;
          ctx.beginPath();
          ctx.ellipse(-3, -wing, 7, 4, 0, 0, TAU);
          ctx.fill();
          ctx.beginPath();
          ctx.ellipse(-3, wing, 7, 4, 0, 0, TAU);
          ctx.fill();
          ctx.fillStyle = "#ffffff";
          ctx.beginPath();
          ctx.arc(0, 0, 3, 0, TAU);
          ctx.fill();
          ctx.restore();
        }

        for (let i = 0; i < tauntTexts.length; i += 1) {
          const taunt = tauntTexts[i];
          const progress = taunt.age / taunt.life;
          const alpha = 1 - progress;
          const fontSize = 18 + progress * 68;

          ctx.save();
          ctx.globalAlpha = alpha;
          ctx.font = "700 " + fontSize + "px -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif";
          ctx.textAlign = "center";
          ctx.textBaseline = "middle";
          ctx.fillStyle = "#fff4cf";
          ctx.shadowColor = "rgba(255, 212, 102, 0.8)";
          ctx.shadowBlur = 16;
          ctx.fillText(taunt.text, taunt.x, taunt.y);
          ctx.restore();
        }

        for (let i = 0; i < forceWaves.length; i += 1) {
          const wave = forceWaves[i];
          const progress = wave.age / wave.life;
          const alpha = Math.max(0, 1 - progress);
          const radius = wave.maxRadius * progress;

          ctx.save();
          ctx.globalAlpha = alpha;
          ctx.strokeStyle = "#fff2a8";
          ctx.lineWidth = 5;
          ctx.shadowColor = "rgba(255, 232, 140, 0.8)";
          ctx.shadowBlur = 18;
          ctx.beginPath();
          ctx.arc(wave.x, wave.y, radius, 0, TAU);
          ctx.stroke();
          ctx.restore();
        }
      }

      function drawEffects() {
        for (let i = 0; i < effects.length; i += 1) {
          const effect = effects[i];
          const progress = effect.age / effect.life;
          const alpha = Math.max(0, 1 - progress);
          const radius = effect.maxRadius * (0.25 + progress * 0.75);

          ctx.save();
          ctx.globalCompositeOperation = "lighter";
          const flash = ctx.createRadialGradient(
            effect.x,
            effect.y,
            0,
            effect.x,
            effect.y,
            radius
          );
          flash.addColorStop(0, "rgba(255, 255, 255, " + alpha * 0.9 + ")");
          flash.addColorStop(0.45, "rgba(255, 150, 55, " + alpha * 0.55 + ")");
          flash.addColorStop(1, "rgba(255, 70, 35, 0)");
          ctx.fillStyle = flash;
          ctx.beginPath();
          ctx.arc(effect.x, effect.y, radius, 0, TAU);
          ctx.fill();
          ctx.restore();
        }

        for (let i = 0; i < particles.length; i += 1) {
          const particle = particles[i];
          const alpha = Math.max(0, 1 - particle.age / particle.life);
          ctx.globalAlpha = alpha;
          ctx.fillStyle = particle.color;
          ctx.beginPath();
          ctx.arc(particle.x, particle.y, particle.radius, 0, TAU);
          ctx.fill();
        }
        ctx.globalAlpha = 1;
      }

      function drawHUD() {
        const barWidth = Math.min(560, W * 0.58);
        const barHeight = 28;
        const x = W / 2 - barWidth / 2;
        const y = 68;
        const ratio = Math.max(0, player.hp / playerMaxHp);

        ctx.fillStyle = "rgba(255, 255, 255, 0.08)";
        ctx.fillRect(x - 4, y - 4, barWidth + 8, barHeight + 8);

        const healthGradient = ctx.createLinearGradient(x, 0, x + barWidth, 0);
        healthGradient.addColorStop(0, "#ff5b7a");
        healthGradient.addColorStop(1, "#ff2d55");
        ctx.fillStyle = healthGradient;
        ctx.fillRect(x, y, barWidth * ratio, barHeight);

        ctx.strokeStyle = "rgba(255, 255, 255, 0.65)";
        ctx.lineWidth = 2;
        ctx.strokeRect(x - 4, y - 4, barWidth + 8, barHeight + 8);

        ctx.fillStyle = "#ffffff";
        ctx.font = "700 15px -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif";
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.fillText(
          Math.ceil(player.hp) + " / " + playerMaxHp,
          W / 2,
          y + barHeight / 2
        );

        // 左上角：计时器和关卡数
        ctx.textAlign = "left";
        ctx.textBaseline = "top";
        ctx.fillStyle = "#ffffff";
        ctx.font = "700 18px -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif";
        ctx.fillText("时间 " + formatTime(levelTimeLeft), 24, 20);

        ctx.fillStyle = "#ffd166";
        ctx.fillText("关卡 " + stage, 24, 46);

        if (flightUnlocked) {
          ctx.fillStyle = flightActive
            ? "#ffd166"
            : flightCooldown <= 0
              ? "#8ef0ff"
              : "rgba(255,255,255,0.78)";
          ctx.font = "700 15px -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif";
          const flightText = flightActive
            ? "飞行 " + Math.max(0, flightTimeRemaining).toFixed(1)
            : flightCooldown <= 0
              ? "飞行 就绪"
              : "飞行 " + Math.ceil(flightCooldown) + "s";
          ctx.fillText(flightText, 24, 72);
        }

        // 血条下方：玩家等级
        ctx.textAlign = "center";
        ctx.fillStyle = "rgba(255, 255, 255, 0.9)";
        ctx.font = "700 15px -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif";
        ctx.fillText("等级 " + playerLevel, W / 2, y + barHeight + 12);

        // 等级下方：蓝色经验条
        const requiredXp = xpRequired(playerLevel);
        const xpRatio = Math.max(0, Math.min(1, experience / requiredXp));
        const xpBarWidth = Math.min(430, barWidth * 0.76);
        const xpBarHeight = 11;
        const xpX = W / 2 - xpBarWidth / 2;
        const xpY = y + barHeight + 23;

        ctx.fillStyle = "rgba(255, 255, 255, 0.08)";
        ctx.fillRect(xpX - 3, xpY - 3, xpBarWidth + 6, xpBarHeight + 6);

        const xpGradient = ctx.createLinearGradient(xpX, 0, xpX + xpBarWidth, 0);
        xpGradient.addColorStop(0, "#57b9ff");
        xpGradient.addColorStop(1, "#2f6fff");
        ctx.fillStyle = xpGradient;
        ctx.fillRect(xpX, xpY, xpBarWidth * xpRatio, xpBarHeight);

        ctx.strokeStyle = "rgba(255, 255, 255, 0.55)";
        ctx.lineWidth = 2;
        ctx.strokeRect(xpX - 3, xpY - 3, xpBarWidth + 6, xpBarHeight + 6);

        ctx.fillStyle = "#ffffff";
        ctx.font = "700 10px -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif";
        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.fillText(
          Math.floor(experience) + " / " + requiredXp,
          W / 2,
          xpY + xpBarHeight / 2
        );

        // 右上角金币数量
        ctx.textAlign = "right";
        ctx.textBaseline = "middle";
        ctx.fillStyle = "#ffd166";
        ctx.beginPath();
        ctx.arc(W - 31, y + 12, 9, 0, TAU);
        ctx.fill();
        ctx.fillStyle = "#f4a62a";
        ctx.beginPath();
        ctx.arc(W - 31, y + 12, 5, 0, TAU);
        ctx.fill();

        ctx.fillStyle = "#ffffff";
        ctx.fillText(coins, W - 16, y + 13);
      }

      function drawGameOver() {
        ctx.fillStyle = "rgba(2, 4, 8, 0.76)";
        ctx.fillRect(0, 0, W, H);

        ctx.textAlign = "center";
        ctx.textBaseline = "middle";
        ctx.fillStyle = "#ffffff";
        ctx.font = "700 38px -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif";
        ctx.fillText("游戏结束", W / 2, H / 2 - 42);

        ctx.fillStyle = "rgba(255, 255, 255, 0.8)";
        ctx.font = "500 18px -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif";
        ctx.fillText("金币 " + coins + "    经验 " + experience, W / 2, H / 2 + 8);

        ctx.fillStyle = "#ffd166";
        ctx.font = "600 17px -apple-system, BlinkMacSystemFont, Segoe UI, sans-serif";
        ctx.fillText("按 Enter 重新开始", W / 2, H / 2 + 54);
      }

      function draw() {
        drawBackground();

        ctx.save();
        ctx.translate(W / 2 - player.x, H / 2 - player.y);

        for (let i = 0; i < items.length; i += 1) {
          drawItem(items[i]);
        }

        drawPathMarks();
        drawFirePatches();

        for (let i = 0; i < monsters.length; i += 1) {
          if (monsters[i].dead) continue;
          drawMonster(monsters[i]);
        }

        for (let i = 0; i < projectiles.length; i += 1) {
          drawProjectile(projectiles[i]);
        }

        drawSkillEffects();
        drawPierceEffects();
        drawEffects();
        drawPlayer();
        ctx.restore();

        drawHUD();

        if (gameOver) {
          drawGameOver();
        }
      }

      function setKey(key, value) {
        if (key === "w") keys.w = value;
        else if (key === "a") keys.a = value;
        else if (key === "s") keys.s = value;
        else if (key === "d") keys.d = value;
        else if (key === " ") keys.space = value;
      }

      function keyName(eventKey) {
        const key = eventKey.toLowerCase();
        if (key === "arrowup") return "w";
        if (key === "arrowdown") return "s";
        if (key === "arrowleft") return "a";
        if (key === "arrowright") return "d";
        return key;
      }

      window.addEventListener("resize", resize);

      window.addEventListener("keydown", (event) => {
        const key = keyName(event.key);

        if (gameOver && key === "enter") {
          event.preventDefault();
          resetGame();
          return;
        }

        if (["w", "a", "s", "d", " "].includes(key)) {
          event.preventDefault();
          setKey(key, true);
        }
      });

      window.addEventListener("keyup", (event) => {
        const key = keyName(event.key);
        if (["w", "a", "s", "d", " "].includes(key)) {
          setKey(key, false);
        }
      });

      canvas.addEventListener("pointerdown", () => {
        if (gameOver) {
          resetGame();
        }
      });

      nextLevelBtn.addEventListener("click", startNextLevel);
      shopRefreshBtn.addEventListener("click", refreshShop);

      window.addEventListener("blur", () => {
        keys.w = false;
        keys.a = false;
        keys.s = false;
        keys.d = false;
        keys.space = false;
      });

      function loop(now) {
        const dt = Math.min((now - last) / 1000, 0.05);
        last = now;

        if (!gameOver && !gamePaused) {
          update(dt);
        }

        draw();
        requestAnimationFrame(loop);
      }

      resize();
      resetGame();
      requestAnimationFrame(loop);
    })();
  </script>
</body>
</html>
