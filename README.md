Anti-Swear
==========
package me.A5H73Y.NoSwear;

import java.io.File;
import java.io.IOException;
import java.io.PrintStream;
import java.util.Arrays;
import java.util.List;
import java.util.logging.Logger;
import org.bukkit.Bukkit;
import org.bukkit.ChatColor;
import org.bukkit.Server;
import org.bukkit.command.Command;
import org.bukkit.command.CommandSender;
import org.bukkit.configuration.file.FileConfiguration;
import org.bukkit.configuration.file.FileConfigurationOptions;
import org.bukkit.configuration.file.YamlConfiguration;
import org.bukkit.entity.Player;
import org.bukkit.plugin.PluginManager;
import org.bukkit.plugin.java.JavaPlugin;

public class NoSwear extends JavaPlugin
{
  public static NoSwearListener plugin;
  public final Logger logger = Logger.getLogger("Minecraft");
  public final NoSwearListener playerListener = new NoSwearListener(this);
  String NoSwearString = ChatColor.BLACK + "[" + ChatColor.RED + "NoSwear" + ChatColor.BLACK + "] " + ChatColor.WHITE;

  public boolean updateavail = false;
  public static File pluginFolder;
  public static File wordFile;
  public static FileConfiguration wordData;
  public String[] muted = new String[0];
  String cmdtarget;
  Player targetPlayer;
  String playerconv;

  public void onDisable()
  {
    this.logger.info("[NoSwear] Disabled!");
  }

  public void onEnable()
  {
    try {
      Metrics metrics = new Metrics(this); metrics.start();
    } catch (IOException e) {
      System.out.println("[NoSwear] Error Submitting stats!");
    }

    PluginManager pm = getServer().getPluginManager();
    pm.registerEvents(this.playerListener, this);

    FileConfiguration config = getConfig();
    String[] Defaults = { "fuck", "shit", "nigger", "asshole", "twat", "whore", "bitch", "cock", "tits", "cunt", "bastard", "dick", "slag", "wank", "prick" };
    config.options().header("NoSwear Config. NOTE: Banning will override Kick. Leave kick = false and ban = false for unlimited warnings");

    config.addDefault("CheckForUpdates", Boolean.valueOf(true));
    config.addDefault("SpamBlocker.Enabled", Boolean.valueOf(true));
    config.addDefault("CapsBlocker.Enabled", Boolean.valueOf(true));
    config.addDefault("CapsBlocker.MinPercent", Integer.valueOf(50));
    config.addDefault("ResetWarnings.onKick", Boolean.valueOf(true));
    config.addDefault("OnJoin.Broadcast", Boolean.valueOf(true));
    config.addDefault("OnJoin.WarningsLeft", Boolean.valueOf(true));
    config.addDefault("OnSwear.Prevent", Boolean.valueOf(true));
    config.addDefault("OnSwear.Ban", Boolean.valueOf(false));
    config.addDefault("OnSwear.Kick", Boolean.valueOf(true));
    config.addDefault("Block.Websites", Boolean.valueOf(true));
    config.addDefault("Block.Strict", Boolean.valueOf(false));
    config.addDefault("Block.Characters", Boolean.valueOf(false));
    config.addDefault("Block.General", Boolean.valueOf(true));
    config.addDefault("Warnings.Default", Integer.valueOf(3));
    config.addDefault("Message.Ban", String.valueOf("Banned for Swearing "));
    config.addDefault("Message.Kick", String.valueOf("Do not swear "));
    config.addDefault("Message.Muted", String.valueOf("You have been muted!"));
    config.addDefault("Message.Warn", String.valueOf("Do not swear "));
    config.addDefault("Message.Join", String.valueOf("Protects this server!"));
    config.addDefault("Message.Spam", String.valueOf("Please do not spam!"));
    config.addDefault("Muted", Arrays.asList(this.muted));
    config.options().copyDefaults(true);
    saveConfig();

    setup();

    wordData.addDefault("BlockedWords", Arrays.asList(Defaults));
    wordData.options().copyDefaults(true);

    saveWords();

    if (config.getBoolean("OnSwear.BanBlock.Chars")) {
      this.logger.info("[NoSwear] Enabled! Config Loaded! Banning!");
    }
    else if (config.getBoolean("OnSwear.Kick"))
      this.logger.info("[NoSwear] Enabled! Config Loaded! Kicking!");
    else {
      this.logger.info("[NoSwear] Enabled! Config Loaded! Warning!");
    }

    if (config.getBoolean("CheckForUpdates"))
      Updater localUpdater = new Updater(this, 38329, getFile(), Updater.UpdateType.DEFAULT, true);
  }

  public boolean onCommand(CommandSender sender, Command cmd, String commandLabel, String[] args)
  {
    readCommand((Player)sender, commandLabel, args);
    return false;
  }
  public boolean readCommand(Player player, String command, String[] args) {
    if ((command.equalsIgnoreCase("ns")) || (command.equalsIgnoreCase("noswear"))) {
      if (args.length >= 1) {
        if (args[0].equalsIgnoreCase("perms")) {
          player.sendMessage(ChatColor.WHITE + "-- " + ChatColor.DARK_AQUA + this.NoSwearString + ChatColor.WHITE + "--");
          player.sendMessage(ChatColor.BLUE + player.getDisplayName());
          if (player.hasPermission("noswear.ignore"))
            player.sendMessage(ChatColor.DARK_AQUA + "Swearing " + ChatColor.AQUA + "[True]");
          else
            player.sendMessage(ChatColor.DARK_AQUA + "Swearing " + ChatColor.AQUA + "[False]");
          if (player.hasPermission("noswear.admin"))
            player.sendMessage(ChatColor.DARK_AQUA + "Admin " + ChatColor.AQUA + "[True]");
          else
            player.sendMessage(ChatColor.DARK_AQUA + "Admin " + ChatColor.AQUA + "[False]");
          player.sendMessage(ChatColor.WHITE + "---- " + ChatColor.AQUA + "v6.1" + ChatColor.WHITE + " ----");
        }
        else if (args[0].equalsIgnoreCase("list")) {
          if (!player.hasPermission("noswear.admin"))
            player.sendMessage(this.NoSwearString + ChatColor.RED + "You do not have permission: Noswear.admin");
          else {
            player.sendMessage(this.NoSwearString + "Blocked Words: " + ChatColor.AQUA + wordData.getStringList("BlockedWords"));
          }
        }
        else if (args[0].equalsIgnoreCase("reload")) {
          if (!player.hasPermission("noswear.admin")) {
            player.sendMessage(this.NoSwearString + ChatColor.RED + "You do not have permission: Noswear.admin");
          } else {
            reloadConfig();
            saveWords();
            player.sendMessage(this.NoSwearString + ChatColor.AQUA + "Config Reloaded!");
          }

        }
        else if (args[0].equalsIgnoreCase("result")) {
          if (!player.hasPermission("noswear.admin")) {
            player.sendMessage(this.NoSwearString + ChatColor.RED + "You do not have permission: Noswear.admin");
          }
          else if (getConfig().getBoolean("OnSwear.Ban")) {
            this.logger.info(this.NoSwearString + ChatColor.WHITE + "Banning on Swear!");
          }
          else if (getConfig().getBoolean("OnSwear.Kick"))
            player.sendMessage(this.NoSwearString + ChatColor.WHITE + "Kicking on Swear!");
          else
            player.sendMessage(this.NoSwearString + ChatColor.WHITE + "Warning on Swear!");
        }
        else if (args[0].equalsIgnoreCase("muted")) {
          if (!player.hasPermission("NoSwear.admin"))
            player.sendMessage(this.NoSwearString + ChatColor.RED + "You do not have permission: Noswear.admin");
          else {
            player.sendMessage(this.NoSwearString + "Muted Players: " + ChatColor.AQUA + getConfig().getStringList("Muted"));
          }

        }
        else if (args[0].equalsIgnoreCase("warns")) {
          Integer RemainingWarns = getRemWarn(player.getPlayer().getName());
          player.sendMessage(this.NoSwearString + "You have " + RemainingWarns + " / " + getConfig().getInt("Warnings.Default") + " Warnings left.");
        }
        else if (args[0].equalsIgnoreCase("cmds")) {
          player.sendMessage("");
          player.sendMessage("-=- " + ChatColor.BOLD + "[NoSwear]" + ChatColor.RESET + " -=-");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "add " + ChatColor.YELLOW + "(word)" + ChatColor.BLACK + " : " + ChatColor.WHITE + "Add the word to the blocked list");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "del " + ChatColor.YELLOW + "(word)" + ChatColor.BLACK + " : " + ChatColor.WHITE + "Delete the word from the blocked list");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "mute " + ChatColor.YELLOW + "(player)" + ChatColor.BLACK + " : " + ChatColor.WHITE + "Add player to mute list");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "unmute " + ChatColor.YELLOW + "(player)" + ChatColor.BLACK + " : " + ChatColor.WHITE + "Remove player from mute list");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "perms" + ChatColor.YELLOW + " : " + ChatColor.WHITE + "Display the players permissions");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "result" + ChatColor.YELLOW + " : " + ChatColor.WHITE + "Display the result when they run out of warnings");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "list" + ChatColor.YELLOW + " : " + ChatColor.WHITE + "Display a list of the blocked words");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "muted " + ChatColor.YELLOW + " : " + ChatColor.WHITE + "Display a list of all the muted users");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "cmds" + ChatColor.YELLOW + " : " + ChatColor.WHITE + "Display the commands list");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "settings" + ChatColor.YELLOW + " : " + ChatColor.WHITE + "Display the NoSwear settings");
          player.sendMessage(ChatColor.DARK_AQUA + "/ns " + ChatColor.AQUA + "warns" + ChatColor.YELLOW + " : " + ChatColor.WHITE + "Display how many warns you have remaining");
          player.sendMessage("-=-=-");
        }
        else if (args[0].equalsIgnoreCase("settings")) {
          if (!player.hasPermission("noswear.admin")) {
            player.sendMessage(this.NoSwearString + ChatColor.RED + "You do not have permission: Noswear.admin");
          } else {
            player.sendMessage("--- NS Settings ---");
            player.sendMessage(ChatColor.DARK_AQUA + "Reset Warns = " + ChatColor.AQUA + getConfig().getBoolean("ResetWarnings.onKick"));
            player.sendMessage(ChatColor.DARK_AQUA + "Warns = " + ChatColor.AQUA + getConfig().getInt("Warnings.Default"));
            player.sendMessage(ChatColor.DARK_AQUA + "Preset = " + ChatColor.AQUA + getConfig().getBoolean("Block.Presets"));
            player.sendMessage(ChatColor.DARK_AQUA + "Websites = " + ChatColor.AQUA + getConfig().getBoolean("Block.Websites"));
            player.sendMessage(ChatColor.DARK_AQUA + "Characters = " + ChatColor.AQUA + getConfig().getBoolean("Block.Characters"));
            player.sendMessage(ChatColor.DARK_AQUA + "Strict = " + ChatColor.AQUA + getConfig().getBoolean("Block.Strict"));
          }
        } else if (args[0].equalsIgnoreCase("add")) {
          if (!player.hasPermission("noswear.admin")) {
            player.sendMessage(this.NoSwearString + ChatColor.RED + "You do not have permission: Noswear.admin");
          }
          else if (args.length == 2) {
            String lowerc = args[1].toLowerCase();
            if (wordData.getStringList("BlockedWords").contains(lowerc)) {
              player.sendMessage(this.NoSwearString + "Word already added!");
            } else {
              int Place = wordData.getStringList("BlockedWords").size();
              if (getConfig().getStringList("BlockedWords").size() == 0) {
                Place = 0;
              }
              List wordlist = wordData.getStringList("BlockedWords");
              wordlist.add(Place, lowerc);
              wordData.set("BlockedWords", wordlist);
              saveWords();
              player.sendMessage(this.NoSwearString + ChatColor.AQUA + lowerc + ChatColor.WHITE + " has been added.");
            }
          } else if (args.length > 2) {
            player.sendMessage(this.NoSwearString + "Too many arguments!");
            player.sendMessage(ChatColor.DARK_AQUA + "/ns add " + ChatColor.AQUA + "<word>");
          } else if (args.length == 1) {
            player.sendMessage(this.NoSwearString + "Not enough arguments!");
            player.sendMessage(ChatColor.DARK_AQUA + "/ns add " + ChatColor.AQUA + "<word>");
          }

        }
        else if (args[0].equalsIgnoreCase("del")) {
          if (!player.hasPermission("noswear.admin")) {
            player.sendMessage(this.NoSwearString + ChatColor.RED + "You do not have permission: Noswear.admin");
          }
          else if (args.length == 2) {
            if (!getConfig().getStringList("BlockedWords").contains(args[1])) {
              player.sendMessage(this.NoSwearString + ChatColor.DARK_AQUA + args[1] + ChatColor.WHITE + " is not in the list!");
            } else {
              int Place = wordData.getStringList("BlockedWords").size();
              if (wordData.getStringList("BlockedWords").size() == 0) {
                Place = 0;
              }
              List wordlist = getConfig().getStringList("BlockedWords");
              wordlist.remove(args[1]);
              wordData.set("BlockedWords", wordlist);
              saveWords();
              player.sendMessage(ChatColor.AQUA + args[1] + ChatColor.WHITE + " has been removed.");
            }

          }
          else if (args.length > 2) {
            player.sendMessage(this.NoSwearString + "Too many arguments!");
            player.sendMessage(ChatColor.DARK_AQUA + "/ns del " + ChatColor.AQUA + "<word>");
          }
          else if (args.length == 1) {
            player.sendMessage(this.NoSwearString + "Not enough arguments!");
            player.sendMessage(ChatColor.DARK_AQUA + "/ns del " + ChatColor.AQUA + "<word>");
          }
        }
        else if (args[0].equalsIgnoreCase("mute")) {
          if (!player.hasPermission("Noswear.admin")) {
            player.sendMessage(this.NoSwearString + ChatColor.RED + "You do not have permission: Noswear.admin");
          }
          else if (args.length == 2) {
            this.cmdtarget = args[1];
            this.targetPlayer = Bukkit.getPlayer(this.cmdtarget);
            this.playerconv = this.targetPlayer.getName();

            if (getConfig().getStringList("Muted").contains(this.playerconv)) {
              player.sendMessage(this.NoSwearString + "Player already muted!");
            } else {
              int Place = getConfig().getStringList("Muted").size();
              if (getConfig().getStringList("Muted").size() == 0) {
                Place = 0;
              }
              List muted = getConfig().getStringList("Muted");
              muted.add(Place, this.playerconv);
              getConfig().set("Muted", muted);
              saveConfig();
              player.sendMessage(this.NoSwearString + ChatColor.AQUA + this.playerconv + ChatColor.WHITE + " has been muted!");
            }
          }
          else if (args.length > 2) {
            player.sendMessage(this.NoSwearString + "Too many arguments!");
            player.sendMessage(ChatColor.DARK_AQUA + "/ns mute " + ChatColor.AQUA + "<player>");
          } else if (args.length == 1) {
            player.sendMessage(this.NoSwearString + "Not enough arguments!");
            player.sendMessage(ChatColor.DARK_AQUA + "/ns mute " + ChatColor.AQUA + "<player>");
          }

        }
        else if (args[0].equalsIgnoreCase("unmute")) {
          if (!player.hasPermission("Noswear.admin")) {
            player.sendMessage(this.NoSwearString + ChatColor.RED + "You do not have permission: Noswear.admin");
          }
          else if (args.length == 2) {
            if (!getConfig().getStringList("Muted").contains(args[1])) {
              player.sendMessage(this.NoSwearString + "Player not muted!");
            } else {
              int Place = getConfig().getStringList("Muted").size();
              if (getConfig().getStringList("Muted").size() == 0) {
                Place = 0;
              }
              List muted = getConfig().getStringList("Muted");
              muted.remove(args[1]);
              getConfig().set("Muted", muted);
              saveConfig();
              player.sendMessage(this.NoSwearString + ChatColor.AQUA + args[1] + ChatColor.WHITE + " was unmuted.");
            }
          }
          else if (args.length > 2) {
            player.sendMessage(this.NoSwearString + "Too many arguments!");
            player.sendMessage(ChatColor.DARK_AQUA + "/ns unmute " + ChatColor.AQUA + "<player>");
          } else if (args.length == 1) {
            player.sendMessage(this.NoSwearString + "Not enough arguments!");
            player.sendMessage(ChatColor.DARK_AQUA + "/ns unmute " + ChatColor.AQUA + "<player>");
          }
        }
        else
        {
          player.sendMessage(this.NoSwearString + "Unknown Argument");
        }
      } else {
        player.sendMessage(ChatColor.DARK_AQUA + "Usage: /Ns " + ChatColor.AQUA + "(Add / Del / Perms / cmds / List)");
        player.sendMessage(ChatColor.DARK_AQUA + "Created by A5H73Y " + ChatColor.AQUA + "V6.1");
      }
    }
    return false;
  }

  public Integer getDefaultWarn()
  {
    Integer defaultWarnings = Integer.valueOf(getConfig().getInt("Warnings.Default"));
    return defaultWarnings;
  }

  public Integer getRemWarn(String PlayerName) {
    Integer warnRemaining = Integer.valueOf(getConfig().getInt("Warned." + PlayerName, getDefaultWarn().intValue()));
    return warnRemaining;
  }
  public void setRemWarn(String PlayerName, Integer warnRemaining) {
    getConfig().set("Warned." + PlayerName, warnRemaining);
    saveConfig();
  }
  public void saveWords() {
    try {
      wordData.save(wordFile); } catch (IOException ex) { ex.printStackTrace(); }
  }

  public void setup() {
    pluginFolder = getDataFolder();

    wordFile = new File(pluginFolder, "words.yml");
    wordData = new YamlConfiguration();

    if (!wordFile.exists()) {
      this.logger.info("[NoSwear] Creating words.yml");
      try {
        wordFile.createNewFile();
        this.logger.info("[NoSwear] Done");
      } catch (Exception ex) {
        ex.printStackTrace();
        this.logger.info("[NoSwear] Failed!");
      }
    }
    try {
      wordData.load(wordFile);
    } catch (Exception ex) {
      ex.printStackTrace();
    }
  }
}
