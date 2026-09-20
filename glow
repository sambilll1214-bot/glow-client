package com.glowclient;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import net.fabricmc.api.ClientModInitializer;
import net.fabricmc.fabric.api.client.event.lifecycle.v1.ClientTickEvents;
import net.fabricmc.fabric.api.client.keybinding.v1.KeyBindingHelper;
import net.fabricmc.fabric.api.client.networking.v1.ClientPlayConnectionEvents;
import net.fabricmc.fabric.api.client.rendering.v1.HudRenderCallback;
import net.fabricmc.fabric.api.client.screen.v1.ScreenEvents;
import net.fabricmc.loader.api.FabricLoader;
import net.minecraft.client.MinecraftClient;
import net.minecraft.client.gui.DrawContext;
import net.minecraft.client.gui.screen.Screen;
import net.minecraft.client.gui.screen.TitleScreen;
import net.minecraft.client.gui.widget.ButtonWidget;
import net.minecraft.client.option.KeyBinding;
import net.minecraft.item.Items;
import net.minecraft.screen.slot.SlotActionType;
import net.minecraft.text.Text;
import net.minecraft.util.Hand;
import net.minecraft.util.hit.BlockHitResult;
import net.minecraft.util.hit.EntityHitResult;
import org.lwjgl.glfw.GLFW;

import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;

public class GlowClient implements ClientModInitializer {
    public static final String NAME = "Glow Client";
    public static final String VERSION = "v2.0 (1.21 PvP Edition)";
    private static KeyBinding guiKeyBinding;

    @Override
    public void onInitializeClient() {
        // 1. Load configuration file
        Config.load();

        // 2. Register Right-Shift for ClickGUI
        guiKeyBinding = KeyBindingHelper.registerKeyBinding(new KeyBinding(
                "Open Glow GUI",
                net.minecraft.client.util.InputUtil.Type.KEYSYM,
                GLFW.GLFW_KEY_RIGHT_SHIFT,
                "Glow Client"
        ));

        // 3. Custom Main Menu / Game Lobby Interceptor
        ScreenEvents.BEFORE_INIT.register((client, screen, scaledWidth, scaledHeight) -> {
            if (screen instanceof TitleScreen && !(screen instanceof GlowMainMenu)) {
                client.setScreen(new GlowMainMenu());
            }
        });

        // 4. Server Join Message
        ClientPlayConnectionEvents.JOIN.register((handler, sender, client) -> {
            if (client.player != null) {
                client.player.sendMessage(Text.literal("§b[" + NAME + "] §fLoaded successfully. Press §eRight Shift §fto open the GUI."), false);
            }
        });

        // 5. On-Screen HUD Watermark
        HudRenderCallback.EVENT.register((drawContext, tickDelta) -> {
            MinecraftClient client = MinecraftClient.getInstance();
            if (client.options.debugEnabled || client.player == null) return;
            drawContext.drawTextWithShadow(client.textRenderer, "§b" + NAME + " §7" + VERSION, 5, 5, 0xFFFFFF);
            
            // Draw active modules list
            int y = 20;
            if (Config.state.crystalTweaks) { drawContext.drawTextWithShadow(client.textRenderer, "§aCrystalSpam", 5, y, 0xFFFFFF); y += 10; }
            if (Config.state.maceTweaks) { drawContext.drawTextWithShadow(client.textRenderer, "§aMaceSmash", 5, y, 0xFFFFFF); y += 10; }
            if (Config.state.swordTweaks) { drawContext.drawTextWithShadow(client.textRenderer, "§aSwordAutoCrit", 5, y, 0xFFFFFF); y += 10; }
            if (Config.state.autoTotem) { drawContext.drawTextWithShadow(client.textRenderer, "§aAutoTotem", 5, y, 0xFFFFFF); y += 10; }
            if (Config.state.fastPlace) { drawContext.drawTextWithShadow(client.textRenderer, "§aFastPlace", 5, y, 0xFFFFFF); y += 10; }
        });

        // 6. Ticks Modules 20 times a second
        ClientTickEvents.END_CLIENT_TICK.register(client -> {
            if (guiKeyBinding.wasPressed()) client.setScreen(new GlowClickGUI());
            if (client.player == null || client.world == null) return;

            if (Config.state.crystalTweaks) Modules.doCrystalSpam(client);
            if (Config.state.maceTweaks) Modules.doMaceTweaks(client);
            if (Config.state.swordTweaks) Modules.doAutoCrit(client);
            if (Config.state.anchorTweaks) Modules.doAnchorTweaks(client);
            if (Config.state.autoTotem) Modules.doAutoTotem(client);
            if (Config.state.fastPlace) Modules.doFastPlace(client);
            if (Config.state.autoSprint) Modules.doAutoSprint(client);
        });
    }

    // ==========================================
    // MODULE LOGIC
    // ==========================================
    public static class Modules {
        public static void doCrystalSpam(MinecraftClient client) {
            if (client.options.useKey.isPressed() && isHolding(client, Items.END_CRYSTAL)) {
                if (client.crosshairTarget instanceof BlockHitResult hit) {
                    Hand hand = client.player.getMainHandStack().isOf(Items.END_CRYSTAL) ? Hand.MAIN_HAND : Hand.OFF_HAND;
                    client.interactionManager.interactBlock(client.player, hand, hit);
                    client.player.swingHand(hand);
                }
            }
        }

        public static void doMaceTweaks(MinecraftClient client) {
            if (client.options.attackKey.isPressed() && isHolding(client, Items.MACE)) {
                // Only swing if falling > 1.5 blocks for huge damage multiplier
                if (client.player.fallDistance > 1.5f && !client.player.isOnGround()) {
                    if (client.crosshairTarget instanceof EntityHitResult hit) {
                        client.interactionManager.attackEntity(client.player, hit.getEntity());
                        client.player.swingHand(Hand.MAIN_HAND);
                        client.options.attackKey.setPressed(false);
                    }
                }
            }
        }

        public static void doAutoCrit(MinecraftClient client) {
            if (client.options.attackKey.isPressed() && isHolding(client, Items.NETHERITE_SWORD, Items.DIAMOND_SWORD)) {
                if (client.player.fallDistance > 0.0f && !client.player.isOnGround() && !client.player.isClimbing()) {
                    if (client.crosshairTarget instanceof EntityHitResult hit) {
                        client.interactionManager.attackEntity(client.player, hit.getEntity());
                        client.player.swingHand(Hand.MAIN_HAND);
                        client.options.attackKey.setPressed(false);
                    }
                }
            }
        }

        public static void doAnchorTweaks(MinecraftClient client) {
            if (client.options.useKey.isPressed() && isHolding(client, Items.GLOWSTONE)) {
                if (client.crosshairTarget instanceof BlockHitResult hit) {
                    client.interactionManager.interactBlock(client.player, Hand.MAIN_HAND, hit);
                    client.player.swingHand(Hand.MAIN_HAND);
                }
            }
        }

        public static void doAutoTotem(MinecraftClient client) {
            if (client.currentScreen != null) return;
            if (client.player.getOffHandStack().getItem() != Items.TOTEM_OF_UNDYING) {
                for (int i = 9; i < 36; i++) {
                    if (client.player.getInventory().getStack(i).getItem() == Items.TOTEM_OF_UNDYING) {
                        client.interactionManager.clickSlot(client.player.playerScreenHandler.syncId, i, 0, SlotActionType.PICKUP, client.player);
                        client.interactionManager.clickSlot(client.player.playerScreenHandler.syncId, 45, 0, SlotActionType.PICKUP, client.player);
                        client.interactionManager.clickSlot(client.player.playerScreenHandler.syncId, i, 0, SlotActionType.PICKUP, client.player);
                        break;
                    }
                }
            }
        }

        public static void toggleFpsBoost(MinecraftClient client, boolean state) {
            if (state) {
                client.options.getGamma().setValue(100.0);
                client.options.getParticles().setValue(net.minecraft.client.option.ParticlesMode.MINIMAL);
                client.options.getSmoothLighting().setValue(false);
            } else {
                client.options.getGamma().setValue(1.0);
                client.options.getParticles().setValue(net.minecraft.client.option.ParticlesMode.ALL);
                client.options.getSmoothLighting().setValue(true);
            }
        }

        public static void doFastPlace(MinecraftClient client) {
            if (client.options.useKey.isPressed() && client.crosshairTarget instanceof BlockHitResult hit) {
                Hand hand = Hand.MAIN_HAND;
                client.interactionManager.interactBlock(client.player, hand, hit);
            }
        }

        public static void doAutoSprint(MinecraftClient client) {
            if (client.player.forwardSpeed > 0 && !client.player.isSneaking() && !client.player.horizontalCollision) {
                client.player.setSprinting(true);
            }
        }

        private static boolean isHolding(MinecraftClient client, net.minecraft.item.Item... items) {
            for (net.minecraft.item.Item i : items) {
                if (client.player.getMainHandStack().isOf(i) || client.player.getOffHandStack().isOf(i)) return true;
            }
            return false;
        }
    }

    // ==========================================
    // CONFIGURATION SAVING (glowclient.json)
    // ==========================================
    public static class Config {
        private static final File FILE = new File(FabricLoader.getInstance().getConfigDir().toFile(), "glowclient.json");
        private static final Gson GSON = new GsonBuilder().setPrettyPrinting().create();

        public static class State {
            public boolean crystalTweaks = false;
            public boolean maceTweaks = false;
            public boolean swordTweaks = false;
            public boolean anchorTweaks = false;
            public boolean fpsBoost = false;
            public boolean autoTotem = false;
            public boolean fastPlace = false;
            public boolean autoSprint = false;
        }

        public static State state = new State();

        public static void load() {
            if (FILE.exists()) {
                try (FileReader reader = new FileReader(FILE)) {
                    state = GSON.fromJson(reader, State.class);
                } catch (IOException e) { e.printStackTrace(); }
            } else { save(); }
        }

        public static void save() {
            try (FileWriter writer = new FileWriter(FILE)) {
                GSON.toJson(state, writer);
            } catch (IOException e) { e.printStackTrace(); }
        }
    }

    // ==========================================
    // GUI / CLICK GUI
    // ==========================================
    public static class GlowClickGUI extends Screen {
        public GlowClickGUI() { super(Text.literal("Glow Client")); }

        @Override
        protected void init() {
            int x = this.width / 2 - 100;
            int y = 40;

            this.addDrawableChild(ButtonWidget.builder(Text.literal("Crystal Tweaks: " + (Config.state.crystalTweaks ? "§aON" : "§cOFF")), b -> {
                Config.state.crystalTweaks = !Config.state.crystalTweaks; Config.save(); b.setMessage(Text.literal("Crystal Tweaks: " + (Config.state.crystalTweaks ? "§aON" : "§cOFF")));
            }).dimensions(x, y, 98, 20).build());

            this.addDrawableChild(ButtonWidget.builder(Text.literal("Mace Smash: " + (Config.state.maceTweaks ? "§aON" : "§cOFF")), b -> {
                Config.state.maceTweaks = !Config.state.maceTweaks; Config.save(); b.setMessage(Text.literal("Mace Smash: " + (Config.state.maceTweaks ? "§aON" : "§cOFF")));
            }).dimensions(x + 102, y, 98, 20).build());

            this.addDrawableChild(ButtonWidget.builder(Text.literal("Sword AutoCrit: " + (Config.state.swordTweaks ? "§aON" : "§cOFF")), b -> {
                Config.state.swordTweaks = !Config.state.swordTweaks; Config.save(); b.setMessage(Text.literal("Sword AutoCrit: " + (Config.state.swordTweaks ? "§aON" : "§cOFF")));
            }).dimensions(x, y += 25, 98, 20).build());

            this.addDrawableChild(ButtonWidget.builder(Text.literal("Anchor Tweaks: " + (Config.state.anchorTweaks ? "§aON" : "§cOFF")), b -> {
                Config.state.anchorTweaks = !Config.state.anchorTweaks; Config.save(); b.setMessage(Text.literal("Anchor Tweaks: " + (Config.state.anchorTweaks ? "§aON" : "§cOFF")));
            }).dimensions(x + 102, y, 98, 20).build());

            this.addDrawableChild(ButtonWidget.builder(Text.literal("Auto Totem: " + (Config.state.autoTotem ? "§aON" : "§cOFF")), b -> {
                Config.state.autoTotem = !Config.state.autoTotem; Config.save(); b.setMessage(Text.literal("Auto Totem: " + (Config.state.autoTotem ? "§aON" : "§cOFF")));
            }).dimensions(x, y += 25, 98, 20).build());

            this.addDrawableChild(ButtonWidget.builder(Text.literal("Fast Place: " + (Config.state.fastPlace ? "§aON" : "§cOFF")), b -> {
                Config.state.fastPlace = !Config.state.fastPlace; Config.save(); b.setMessage(Text.literal("Fast Place: " + (Config.state.fastPlace ? "§aON" : "§cOFF")));
            }).dimensions(x + 102, y, 98, 20).build());

            this.addDrawableChild(ButtonWidget.builder(Text.literal("Auto Sprint: " + (Config.state.autoSprint ? "§aON" : "§cOFF")), b -> {
                Config.state.autoSprint = !Config.state.autoSprint; Config.save(); b.setMessage(Text.literal("Auto Sprint: " + (Config.state.autoSprint ? "§aON" : "§cOFF")));
            }).dimensions(x, y += 25, 98, 20).build());

            this.addDrawableChild(ButtonWidget.builder(Text.literal("Fullbright & FPS: " + (Config.state.fpsBoost ? "§aON" : "§cOFF")), b -> {
                Config.state.fpsBoost = !Config.state.fpsBoost; Config.save(); Modules.toggleFpsBoost(MinecraftClient.getInstance(), Config.state.fpsBoost);
                b.setMessage(Text.literal("Fullbright & FPS: " + (Config.state.fpsBoost ? "§aON" : "§cOFF")));
            }).dimensions(x + 102, y, 98, 20).build());
        }

        @Override
        public void render(DrawContext context, int mouseX, int mouseY, float delta) {
            this.renderBackground(context, mouseX, mouseY, delta);
            context.fill(this.width / 2 - 120, 20, this.width / 2 + 120, this.height - 20, 0x88000000);
            context.drawCenteredTextWithShadow(this.textRenderer, "§b§l" + NAME + " §7- Settings", this.width / 2, 25, 0xFFFFFF);
            super.render(context, mouseX, mouseY, delta);
        }
    }

    // ==========================================
    // CUSTOM LOBBY (MAIN MENU)
    // ==========================================
    public static class GlowMainMenu extends Screen {
        public GlowMainMenu() { super(Text.literal("Glow Client Menu")); }

        @Override
        protected void init() {
            int x = this.width / 2 - 100;
            int y = this.height / 2 - 10;
            this.addDrawableChild(ButtonWidget.builder(Text.literal("Singleplayer"), b -> this.client.setScreen(new net.minecraft.client.gui.screen.world.SelectWorldScreen(this))).dimensions(x, y, 200, 20).build());
            this.addDrawableChild(ButtonWidget.builder(Text.literal("Multiplayer"), b -> this.client.setScreen(new net.minecraft.client.gui.screen.multiplayer.MultiplayerScreen(this))).dimensions(x, y += 25, 200, 20).build());
            this.addDrawableChild(ButtonWidget.builder(Text.literal("Options"), b -> this.client.setScreen(new net.minecraft.client.gui.screen.option.OptionsScreen(this, this.client.options))).dimensions(x, y += 25, 98, 20).build());
            this.addDrawableChild(ButtonWidget.builder(Text.literal("Quit Game"), b -> this.client.scheduleStop()).dimensions(x + 102, y, 98, 20).build());
        }

        @Override
        public void render(DrawContext context, int mouseX, int mouseY, float delta) {
            context.fill(0, 0, this.width, this.height, 0xFF0D0D11); // Dark background
            context.drawCenteredTextWithShadow(this.textRenderer, "§b§l" + NAME, this.width / 2, this.height / 2 - 50, 0x00FFFF);
            context.drawCenteredTextWithShadow(this.textRenderer, "§7Advanced 1.21.x PvP Modification", this.width / 2, this.height / 2 - 35, 0xFFFFFF);
            super.render(context, mouseX, mouseY, delta);
        }
    }
}
