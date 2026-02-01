---
layout: post
title: "记一次 Bug 修复的经历"
date: 2026-02-01 17:13:00 +0800
categories: [开发]
math: false
mermaid: false
pin: false
tags: [Java, 面向对象, Minecraft, Slimefun4, 插件, bug]
---

## 背景

今天刚起来，就看到邮箱里有一条新邮件，是 GitHub 发来的一封通知：

> **[CrispyXYZ/GeneticChickengineering-CN-RC30-Extended] 1.21.11 papererror (Issue #14)**  
>   
> **xinge54088** created an issue [(CrispyXYZ/GeneticChickengineering-CN-RC30-Extended#14)](https://github.com/CrispyXYZ/GeneticChickengineering-CN-RC30-Extended/issues/14)  
> [01:31:18 ERROR]: [GeneticChickengineering] Item "GCE_POCKET_CHICKEN" from GeneticChickengineering vBuild 10 (git da9af98) has caused an Error!  
> [01:31:18 ERROR]: [GeneticChickengineering] You can report it here: https://github.com/CrispyXYZ/GeneticChickengineering-CN-RC30-Extended/issues  
> [01:31:18 ERROR]: [GeneticChickengineering] Could not pass "ItemUseHandler" for PocketChicken - 'GCE_POCKET_CHICKEN' (GeneticChickengineering vBuild 10 (git da9af98))

原来是一个旧项目不兼容 Minecraft 1.21.11 的新版本。

## 解决过程

### 代码分析

老规矩，先 clone 到本地看看能否复现。由于本地还有缓存，所以即使不改 `pom.xml` 也能跑起来。为了跟用户保持相同的环境，就不修改 `pom.xml` 直接执行 Maven 的 `package` 命令。把构建的 JAR 包丢进服务端插件列表，上线测试看看。

```log
PS D:\mcserver-test> java -jar .\paper-1.21.11-100.jar
Starting org.bukkit.craftbukkit.Main
[11:17:19 INFO]: [bootstrap] Running Java 21 (OpenJDK 64-Bit Server VM 21.0.4+7-LTS; Azul Systems, Inc. Zulu21.36+17-CA) on Windows 11 10.0 (amd64)
[11:17:19 INFO]: [bootstrap] Loading Paper 1.21.11-100-main@4873e3f (2026-01-29T11:02:49Z) for Minecraft 1.21.11
[11:17:19 INFO]: [PluginInitializerManager] Initializing plugins...
[11:17:20 INFO]: [PluginRemapper] Remapping plugin 'plugins\GeneticChickengineering v1.2.1.jar'...
[11:17:20 INFO]: [PluginRemapper] Remapping plugin 'plugins\Slimefun-2025.11-release.jar'...
[11:17:23 INFO]: [PluginRemapper] Done remapping plugin 'plugins\Slimefun-2025.11-release.jar' in 3113ms.
[11:17:24 INFO]: [PluginRemapper] Done remapping plugin 'plugins\GeneticChickengineering v1.2.1.jar' in 3818ms.
[11:17:24 INFO]: [PluginInitializerManager] Initialized 2 plugins
[11:17:24 INFO]: [PluginInitializerManager] Bukkit plugins (2):
 - GeneticChickengineering (1.2.1), Slimefun (2025.11-release)
......
......
[11:21:00 INFO]: CrispyXYZ issued server command: /gamemode creative
[11:21:00 INFO]: [CrispyXYZ: Set own game mode to Creative Mode]
[11:21:23 INFO]: CrispyXYZ issued server command: /sf cheat
[11:23:22 ERROR]: [GeneticChickengineering] Item "GCE_POCKET_CHICKEN" from GeneticChickengineering v1.2.1 has caused an Error!
[11:23:22 ERROR]: [GeneticChickengineering] You can report it here: https://github.com/CrispyXYZ/GeneticChickengineering-CN-RC30-Extended/issues
[11:23:22 ERROR]: [GeneticChickengineering] Could not pass "ItemUseHandler" for PocketChicken - 'GCE_POCKET_CHICKEN' (GeneticChickengineering v1.2.1)
java.lang.NullPointerException: Cannot invoke "org.bukkit.NamespacedKey.getNamespace()" because "key" is null
        at org.bukkit.craftbukkit.util.CraftNamespacedKey.toMinecraft(CraftNamespacedKey.java:23) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at org.bukkit.craftbukkit.util.CraftNamespacedKey.toResourceKey(CraftNamespacedKey.java:34) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at org.bukkit.craftbukkit.CraftRegistry.get(CraftRegistry.java:199) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at org.bukkit.craftbukkit.CraftRegistry.get(CraftRegistry.java:132) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at org.bukkit.craftbukkit.util.CraftMagicNumbers.get(CraftMagicNumbers.java:485) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at org.bukkit.attribute.Attribute.valueOf(Attribute.java:180) ~[paper-api-1.21.11-R0.1-SNAPSHOT.jar:?]
        at org.bukkit.craftbukkit.legacy.FieldRename.valueOf_Attribute(FieldRename.java:416) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at GeneticChickengineering v1.2.1.jar//space.kiichan.geneticchickengineering.adapter.MobAdapter.BUKKIT_CUSTOM_METHOD_org_bukkit_craftbukkit_legacy_FieldRename_valueOf_Attribute(MobAdapter.java) ~[?:?]
        at GeneticChickengineering v1.2.1.jar//space.kiichan.geneticchickengineering.adapter.MobAdapter.apply(MobAdapter.java:84) ~[?:?]
        at GeneticChickengineering v1.2.1.jar//space.kiichan.geneticchickengineering.adapter.AnimalsAdapter.apply(AnimalsAdapter.java:53) ~[?:?]
        at GeneticChickengineering v1.2.1.jar//space.kiichan.geneticchickengineering.chickens.PocketChicken.lambda$getItemHandler$0(PocketChicken.java:251) ~[?:?]
        at Slimefun-2025.11-release.jar//io.github.thebusybiscuit.slimefun4.implementation.listeners.SlimefunItemInteractListener.lambda$rightClickItem$0(SlimefunItemInteractListener.java:104) ~[?:?]
        at Slimefun-2025.11-release.jar//io.github.thebusybiscuit.slimefun4.api.items.SlimefunItem.callItemHandler(SlimefunItem.java:1004) ~[?:?]
        at Slimefun-2025.11-release.jar//io.github.thebusybiscuit.slimefun4.implementation.listeners.SlimefunItemInteractListener.rightClickItem(SlimefunItemInteractListener.java:104) ~[?:?]
        at Slimefun-2025.11-release.jar//io.github.thebusybiscuit.slimefun4.implementation.listeners.SlimefunItemInteractListener.onRightClick(SlimefunItemInteractListener.java:73) ~[?:?]
        at co.aikar.timings.TimedEventExecutor.execute(TimedEventExecutor.java:80) ~[paper-api-1.21.11-R0.1-SNAPSHOT.jar:?]
        at org.bukkit.plugin.RegisteredListener.callEvent(RegisteredListener.java:71) ~[paper-api-1.21.11-R0.1-SNAPSHOT.jar:?]
        at io.papermc.paper.plugin.manager.PaperEventManager.callEvent(PaperEventManager.java:54) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at io.papermc.paper.plugin.manager.PaperPluginManagerImpl.callEvent(PaperPluginManagerImpl.java:131) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at org.bukkit.plugin.SimplePluginManager.callEvent(SimplePluginManager.java:628) ~[paper-api-1.21.11-R0.1-SNAPSHOT.jar:?]
        at org.bukkit.craftbukkit.event.CraftEventFactory.callPlayerInteractEvent(CraftEventFactory.java:654) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at net.minecraft.server.level.ServerPlayerGameMode.useItemOn(ServerPlayerGameMode.java:507) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at net.minecraft.server.network.ServerGamePacketListenerImpl.handleUseItemOn(ServerGamePacketListenerImpl.java:2094) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at net.minecraft.network.protocol.game.ServerboundUseItemOnPacket.handle(ServerboundUseItemOnPacket.java:45) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at net.minecraft.network.protocol.game.ServerboundUseItemOnPacket.handle(ServerboundUseItemOnPacket.java:10) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at net.minecraft.network.PacketProcessor$ListenerAndPacket.handle(PacketProcessor.java:99) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at net.minecraft.network.PacketProcessor.executeSinglePacket(PacketProcessor.java:33) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at net.minecraft.server.MinecraftServer.pollTask(MinecraftServer.java:1504) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at net.minecraft.server.MinecraftServer.recordTaskExecutionTimeWhileWaiting(MinecraftServer.java:1230) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at net.minecraft.server.MinecraftServer.runServer(MinecraftServer.java:1346) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at net.minecraft.server.MinecraftServer.lambda$spin$2(MinecraftServer.java:388) ~[paper-1.21.11.jar:1.21.11-100-4873e3f]
        at java.base/java.lang.Thread.run(Thread.java:1583) ~[?:?]
```

果然，在本地运行时，可以复现这个问题。在右键使用物品 `GCE_POCKET_CHICKEN` （装有鸡的袋子） 时，服务端恰好抛出上述错误。我们提取一下关键信息：

- `java.lang.NullPointerException` - 空指针异常：指引我们应该检查在代码中是否存在 `null` 情况。据此推测，此 bug 极有可能是 ~~bugjump~~ mojang 对原版游戏的修改导致了插件获取某些值时出现了 `null` 。
- `at GeneticChickengineering v1.2.1.jar//space.kiichan.geneticchickengineering.adapter.MobAdapter.apply(MobAdapter.java:84)` ： 指引我们应该关注 `MobAdapter.apply` 方法，在 `MobAdapter.java` 文件的第 84 行。

直接看源代码：

```java
79 default void apply(T entity, JsonObject json) {
80     // We need to apply Attributes before the health.
81     JsonObject attributes = json.getAsJsonObject("_attributes");
82
83     for (Map.Entry<String, JsonElement> entry : attributes.entrySet()) {
84         AttributeInstance instance = entity.getAttribute(Attribute.valueOf(entry.getKey()));
```

嗯？这也能抛空指针异常？！难道是 `Attribute.valueOf` 方法返回 `null` 了？尝试跳转到 `Attribute.valueOf` 却提示无法下载源文件，只能看反编译版本。反编译结果显示， `Attribute` 是一个枚举类，枚举类有一个内置的静态方法 `valueOf` ，用于将字符串转换为枚举类型，若字符串不存在，则抛出 `IllegalArgumentException` 异常。回头看看堆栈，`at org.bukkit.attribute.Attribute.valueOf(Attribute.java:180)`，可是反编译文件也只有不到80行……换个思路吧，去看看游戏数据。

### 游戏数据分析

```text
> /data get block 66 63 52
[11:49:34 INFO]: 66, 63, 52 has the following block data: 
{
  z: 52,
  id: "minecraft:chest",
  y: 63,
  x: 66,
  Items: [{
    Slot: 0b,
    id: "minecraft:player_head",
    count: 1,
    components: {
      "minecraft:profile": {
        name: "CS-CoreLib",
        properties: [{
          name: "textures",
          value: "eyJ0ZXh0dXJlcyI6eyJTS0lOIjp7InVybCI6Imh0dHA6Ly90ZXh0dXJlcy5taW5lY3JhZnQubmV0L3RleHR1cmUvMTYzODQ2OWE1OTljZWVmNzIwNzUzNzYwMzI0OGE5YWIxMWZmNTkxZmQzNzhiZWE0NzM1YjM0NmE3ZmFlODkzIn19fQ=="
        }],
        id: [I;707125077, -1632028922, -1392419545, -798288189]
      },
      "minecraft:lore": ["", {
        extra: [{
          italic: 0b,
          underlined: 0b,
          bold: 0b,
          color: "gray",
          obfuscated: 0b,
          strikethrough: 0b,
          text: "血量: "
        }, {
          italic: 0b,
          text: "4.0",
          color: "green"
        }],
        text: ""
      }],
      "minecraft:custom_name": {
        extra: [{
          italic: 0b,
          text: "装有鸡的袋子",
          color: "aqua"
        }],
        text: ""
      },
      "minecraft:custom_data": {
        PublicBukkitValues: {
          "geneticchickengineering:gce_pocket_chicken_dna": [I;3, 3, 3, 1, 3, 3, 0],
          "slimefun:slimefun_item": "GCE_POCKET_CHICKEN",
          "geneticchickengineering:gce_pocket_chicken_adapter": '{"_type":"CHICKEN","_health":4.0,"_absorption":0.0,"_removeWhenFarAway":true,"_customName":null,"_customNameVisible":false,"_ai":true,"_silent":false,"_glowing":false,"_invulnerable":false,"_collidable":true,"_gravity":true,"_fireTicks":0,"_attributes":{"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:fall_damage_multiplier]=net.minecraft.world.entity.ai.attributes.RangedAttribute@66ec58e2}}":{"base":1.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:follow_range]=net.minecraft.world.entity.ai.attributes.RangedAttribute@6d7a8aee}}":{"base":16.0,"modifiers":[{"amount":"-0.02429296946451068","operation":"1","key":"minecraft:random_spawn_bonus"}]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:movement_speed]=net.minecraft.world.entity.ai.attributes.RangedAttribute@707cadcd}}":{"base":0.25,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:max_health]=net.minecraft.world.entity.ai.attributes.RangedAttribute@1bcf1437}}":{"base":4.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:oxygen_bonus]=net.minecraft.world.entity.ai.attributes.RangedAttribute@7d280ac0}}":{"base":0.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:armor_toughness]=net.minecraft.world.entity.ai.attributes.RangedAttribute@758f34ae}}":{"base":0.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:gravity]=net.minecraft.world.entity.ai.attributes.RangedAttribute@5e93a1a4}}":{"base":0.08,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:camera_distance]=net.minecraft.world.entity.ai.attributes.RangedAttribute@789bda7b}}":{"base":4.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:burning_time]=net.minecraft.world.entity.ai.attributes.RangedAttribute@915ea98}}":{"base":1.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:max_absorption]=net.minecraft.world.entity.ai.attributes.RangedAttribute@735359b1}}":{"base":0.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:movement_efficiency]=net.minecraft.world.entity.ai.attributes.RangedAttribute@23e9a2e4}}":{"base":0.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:attack_knockback]=net.minecraft.world.entity.ai.attributes.RangedAttribute@6ef41253}}":{"base":0.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:knockback_resistance]=net.minecraft.world.entity.ai.attributes.RangedAttribute@4773081a}}":{"base":0.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:armor]=net.minecraft.world.entity.ai.attributes.RangedAttribute@28786878}}":{"base":0.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:safe_fall_distance]=net.minecraft.world.entity.ai.attributes.RangedAttribute@13360552}}":{"base":3.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:jump_strength]=net.minecraft.world.entity.ai.attributes.RangedAttribute@21b78c9c}}":{"base":0.41999998688697815,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:water_movement_efficiency]=net.minecraft.world.entity.ai.attributes.RangedAttribute@122508fe}}":{"base":0.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:waypoint_transmit_range]=net.minecraft.world.entity.ai.attributes.RangedAttribute@1db4a1ca}}":{"base":0.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:tempt_range]=net.minecraft.world.entity.ai.attributes.RangedAttribute@1a47649b}}":{"base":10.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:explosion_knockback_resistance]=net.minecraft.world.entity.ai.attributes.RangedAttribute@329b9673}}":{"base":0.0,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:step_height]=net.minecraft.world.entity.ai.attributes.RangedAttribute@2cf1d4da}}":{"base":0.6,"modifiers":[]},"CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:scale]=net.minecraft.world.entity.ai.attributes.RangedAttribute@7ebb89ac}}":{"base":1.0,"modifiers":[]}},"_effects":{},"_scoreboardTags":[],"baby":false,"_age":0,"_ageLock":false,"_breedable":true,"_loveModeTicks":0}'
        }
      }
    }
  }],
  components: {}
}
```

注意看这个属性 `Items[0].components.minecraft:custom_data.PublicBukkitValues.geneticchickengineering:gce_pocket_chicken_adapter`，里面存放的正是我们在 Java 代码中要读取的 JSON 数据，把它格式化方便观察：

```json
{
  "_type": "CHICKEN",
  "_health": 4,
  "_absorption": 0,
  "_removeWhenFarAway": true,
  "_customName": null,
  "_customNameVisible": false,
  "_ai": true,
  "_silent": false,
  "_glowing": false,
  "_invulnerable": false,
  "_collidable": true,
  "_gravity": true,
  "_fireTicks": 0,
  "_attributes": {
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:fall_damage_multiplier]=net.minecraft.world.entity.ai.attributes.RangedAttribute@66ec58e2}}": {
      "base": 1,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:follow_range]=net.minecraft.world.entity.ai.attributes.RangedAttribute@6d7a8aee}}": {
      "base": 16,
      "modifiers": [
        {
          "amount": "-0.02429296946451068",
          "operation": "1",
          "key": "minecraft:random_spawn_bonus"
        }
      ]
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:movement_speed]=net.minecraft.world.entity.ai.attributes.RangedAttribute@707cadcd}}": {
      "base": 0.25,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:max_health]=net.minecraft.world.entity.ai.attributes.RangedAttribute@1bcf1437}}": {
      "base": 4,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:oxygen_bonus]=net.minecraft.world.entity.ai.attributes.RangedAttribute@7d280ac0}}": {
      "base": 0,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:armor_toughness]=net.minecraft.world.entity.ai.attributes.RangedAttribute@758f34ae}}": {
      "base": 0,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:gravity]=net.minecraft.world.entity.ai.attributes.RangedAttribute@5e93a1a4}}": {
      "base": 0.08,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:camera_distance]=net.minecraft.world.entity.ai.attributes.RangedAttribute@789bda7b}}": {
      "base": 4,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:burning_time]=net.minecraft.world.entity.ai.attributes.RangedAttribute@915ea98}}": {
      "base": 1,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:max_absorption]=net.minecraft.world.entity.ai.attributes.RangedAttribute@735359b1}}": {
      "base": 0,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:movement_efficiency]=net.minecraft.world.entity.ai.attributes.RangedAttribute@23e9a2e4}}": {
      "base": 0,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:attack_knockback]=net.minecraft.world.entity.ai.attributes.RangedAttribute@6ef41253}}": {
      "base": 0,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:knockback_resistance]=net.minecraft.world.entity.ai.attributes.RangedAttribute@4773081a}}": {
      "base": 0,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:armor]=net.minecraft.world.entity.ai.attributes.RangedAttribute@28786878}}": {
      "base": 0,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:safe_fall_distance]=net.minecraft.world.entity.ai.attributes.RangedAttribute@13360552}}": {
      "base": 3,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:jump_strength]=net.minecraft.world.entity.ai.attributes.RangedAttribute@21b78c9c}}": {
      "base": 0.41999998688697815,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:water_movement_efficiency]=net.minecraft.world.entity.ai.attributes.RangedAttribute@122508fe}}": {
      "base": 0,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:waypoint_transmit_range]=net.minecraft.world.entity.ai.attributes.RangedAttribute@1db4a1ca}}": {
      "base": 0,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:tempt_range]=net.minecraft.world.entity.ai.attributes.RangedAttribute@1a47649b}}": {
      "base": 10,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:explosion_knockback_resistance]=net.minecraft.world.entity.ai.attributes.RangedAttribute@329b9673}}": {
      "base": 0,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:step_height]=net.minecraft.world.entity.ai.attributes.RangedAttribute@2cf1d4da}}": {
      "base": 0.6,
      "modifiers": []
    },
    "CraftAttribute{holder=Reference{ResourceKey[minecraft:attribute / minecraft:scale]=net.minecraft.world.entity.ai.attributes.RangedAttribute@7ebb89ac}}": {
      "base": 1,
      "modifiers": []
    }
  },
  "_effects": {},
  "_scoreboardTags": [],
  "baby": false,
  "_age": 0,
  "_ageLock": false,
  "_breedable": true,
  "_loveModeTicks": 0
}
```

展开后就能发现问题所在了：`_attributes` 对象里的键格式显然不正确，现在储存的形式更像是某个 `toString()` 方法的返回值，这提示我们应该从储存数据的方法入手。不过，在修改代码之前，先让我去 GitHub 上查阅其它项目是否在 `Attribute` 类上遇到了同样的情况。

### 着手解决问题

经过一番搜索，我发现了有一个工具类的方法这样写：

```java
    /**
     * 获取最大生命值属性。
     * 在 1.21.3 之前，使用 GENERIC_MAX_HEALTH。
     *
     * @return 最大生命值属性
     */
    public static Attribute getMaxHealth() {
        if (SlimefunExtended.getMinecraftVersion().isAtLeast(1, 21, 3)) {
            return Registry.ATTRIBUTE.get(NamespacedKey.fromString("max_health"));
        } else {
            return Attribute.valueOf("GENERIC_MAX_HEALTH");
        }
    }
```

这提示我们在 `1.21.3` 版本前后对属性(attribute)的处理存在差异。因此，让我们更新 `pom.xml` 并对代码进行微调，让整个项目能在 `1.21.11` 的版本跑起来，提交 [commit `558af39`](https://github.com/CrispyXYZ/GeneticChickengineering-CN-RC30-Extended/commit/558af395ccc453b17a7ae0d897c967c9c5b319d9)

依赖库更新后，再回到 `MobAdapter` 文件，这时已经提示我过时的用法了，按照给出的推荐方法，对读取方法进行修改如下：

```java
            NamespacedKey namespacedKey = NamespacedKey.fromString(entry.getKey());
            Attribute attribute = Registry.ATTRIBUTE.get(namespacedKey);
            AttributeInstance instance = entity.getAttribute(attribute);
```

写入方法同样按照注释推荐写法修改。改完后打包上线测试，果然能顺利处理新版的属性数据。但此时无法读取在修改前就储存的数据。

### 向后兼容以及程序的健壮性

为增强程序的健壮性，考虑用户体验，对于旧版产生的数据，应当能在新版正常读取，对于无法读取的数据，应当报一个警告。因此，在读取方法中加入 `null` 判定：

```java
            NamespacedKey namespacedKey = NamespacedKey.fromString(entry.getKey());
            if (namespacedKey == null) {
                namespacedKey = convertOldAttribute(entry.getKey());
            }
            if (namespacedKey == null) {
                getLogger().warning("Could not convert " + entry.getKey() + " to a namespaced key, skipping it.");
                continue;
            }

            Attribute attribute = Registry.ATTRIBUTE.get(namespacedKey);
            if (attribute == null) {
                getLogger().warning("Unrecognizable attribute " + namespacedKey.asString() + ", skipping it.");
                continue;
            }

            AttributeInstance instance = entity.getAttribute(attribute);
```

然后实现旧数据转换方法：

```java
    private NamespacedKey convertOldAttribute(String oldAttribute) {
        String inner = oldAttribute.substring(oldAttribute.indexOf('[')+1, oldAttribute.lastIndexOf(']'));
        if (inner.contains("]")) {
            getLogger().warning("Conversion failed. Old attribute: " + oldAttribute);
            return null;
        }
        String newAttribute = inner.substring(inner.indexOf('/')+1).trim();
        return NamespacedKey.fromString(newAttribute);
    }
```

上线测试，用旧版数据测试成功。再制造异常数据尝试，成功报出警告：

```text
[14:27:43 INFO]: [@: Modified block data of 66, 63, 52]
[14:27:47 WARN]: [GeneticChickengineering] Unrecognizable attribute minecraft:cameaaara_distance, skipping it.
```

至此，bug 修复完成！提交 [commit `2e06a44`](https://github.com/CrispyXYZ/GeneticChickengineering-CN-RC30-Extended/commit/2e06a4479d713021e7c1bf672b2e5e656f5fce3e)。