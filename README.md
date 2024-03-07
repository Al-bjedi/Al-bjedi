const express = require('express');
const app = express();


app.get('/', function (request, response) {
  response.sendFile(__dirname + '/index.html');
});

app.use('/ping', (req, res) => {
  res.send(new Date());
});

app.listen(9080, () => {
  console.log(('Express is ready.').blue.bold)
});

const { Client, Collection, Partials, GatewayIntentBits, ActionRowBuilder, ButtonBuilder, ButtonStyle, Events, EmbedBuilder } = require('discord.js');

const config = require("./config.json");
const { glob } = require("glob");
const { promisify } = require("util");
const { joinVoiceChannel } = require('@discordjs/voice');
const { REST } = require('@discordjs/rest');
const { Routes } = require('discord-api-types/v9');
const { SlashCommandBuilder } = require('@discordjs/builders');
const db = require('quick.db');
const colors = require("colors");

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMembers,
    GatewayIntentBits.GuildEmojisAndStickers,
    GatewayIntentBits.GuildIntegrations,
    GatewayIntentBits.GuildWebhooks,
    GatewayIntentBits.GuildInvites,
    GatewayIntentBits.GuildVoiceStates,
    GatewayIntentBits.GuildPresences,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.GuildMessageReactions,
    GatewayIntentBits.GuildMessageTyping,
    GatewayIntentBits.DirectMessages,
    GatewayIntentBits.DirectMessageReactions,
    GatewayIntentBits.DirectMessageTyping,
    GatewayIntentBits.MessageContent
  ],
  partials: [
    Partials.Message,
    Partials.Channel,
    Partials.GuildMember,
    Partials.Reaction,
    Partials.GuildScheduledEvent,
    Partials.User,
    Partials.ThreadMember
  ],
  shards: "auto",
  allowedMentions: {
    parse: [],
    repliedUser: false
  },
})

client.setMaxListeners(25);
require('events').defaultMaxListeners = 25;

const { createLogger, transports, format } = require('winston');
const path = require('path');

const logger = createLogger({
  level: 'error',
  format: format.combine(
    format.timestamp(),
    format.json(),
  ),
  transports: [
    new transports.File({ filename: path.join(__dirname, 'Logs', 'Errors.json') }),
  ],
});

client.on('error', error => {
  console.error('Discord.js error:', error);
  logger.error('Discord.js error:', error);
});

client.on('warn', warning => {
  console.warn('Discord.js warning:', warning);
});

let antiCrashLogged = false;

process.on('unhandledRejection', (reason, p) => {
  if (!antiCrashLogged) {
    console.error('[antiCrash] :: Unhandled Rejection/Catch');
    console.error(reason, p);
    logger.error('[antiCrash] :: Unhandled Rejection/Catch', { reason, p });
    antiCrashLogged = true;
  }
});

process.on('uncaughtException', (err, origin) => {
  if (!antiCrashLogged) {
    console.error('[antiCrash] :: Uncaught Exception/Catch');
    console.error(err, origin);
    logger.error('[antiCrash] :: Uncaught Exception/Catch', { err, origin });
    antiCrashLogged = true;
  }
});

process.on('uncaughtExceptionMonitor', (err, origin) => {
  if (!antiCrashLogged) {
    console.error('[antiCrash] :: Uncaught Exception/Catch (MONITOR)');
    console.error(err, origin);
    logger.error('[antiCrash] :: Uncaught Exception/Catch (MONITOR)', { err, origin });
    antiCrashLogged = true;
  }
});


module.exports = client;
client.commands = new Collection();
client.events = new Collection();
client.slashCommands = new Collection();
['commands', 'events', 'slash'].forEach(handler => {
  require(`./handlers/`)(client);
})

const commands = client.slashCommands.map(({ execute, ...data }) => data);

// Register slash commands


setTimeout(() => {
  if (!client || !client.user) {
    console.log("Client Not Login, Process Kill")
    process.kill(1);
  } else {
    console.log("Client Login")
  }
}, 5 * 1000 * 60);

client.login(config.token || process.env.token).then((bot)=>{

const rest = new REST({ version: '9' }).setToken(config.token || process.env.token);
rest.put(
  Routes.applicationGuildCommands(config.clientID, config.guildID),
  { body: commands },
).then(() => console.log('Successfully registered application commands.'))
  .catch(console.error)
  
}).catch((err) => {
  console.log(err.message)
})


const { ActionRowBuilder, ButtonBuilder, ButtonStyle, Events, EmbedBuilder } = require("discord.js");
const { glob } = require("glob");
const { promisify } = require("util");
const { prefix } = require('../../config.json');
const { Utils } = require("devtools-ts");
const utilites = new Utils();


module.exports = {
    name: "help",
    description: 'Feeling lost?',
    async execute(client, interaction) {
        try {
            const globPromise = promisify(glob);
            const commandFiles = await globPromise(`/SlashCommands/music/**/*.js`);

            let embed = new EmbedBuilder()

                .setThumbnail(client.user.displayAvatarURL({ dynamic: true }))
            /*
                            .addFields(
                                { name: `/247`, value: `Toggles the 24/7 mode. This makes the bot doesn't leave the voice channel until you stop it.`, inline: false },
                                { name: `/autoplay`, value: `Toggles autoplay for the current guild.`, inline: false },
                                { name: `/join`, value: `join the voice channel.`, inline: false },
                                { name: `/jump`, value: `Skip to a song in the queue.`, inline: false },
                                { name: `/leave`, value: `leave the voice channel.`, inline: false },
                                { name: `/loop`, value: `Toggles the repeat mode.`, inline: false },
                                { name: `/lyrics`, value: `Display lyrics of a song`, inline: false },
                                { name: `/nowplaying`, value: `Shows what is song that the bot is currently playing.`, inline: false },
                                { name: `/pause`, value: `Pauses the currently playing track.`, inline: false },
                                { name: `/play`, value: `Add a song to queue and plays it.`, inline: false },
                                { name: `/previous`, value: `Plays the previous song in the queue.`, inline: false },
                                { name: `/queue`, value: `Display the queue of the current tracks in the playlist.`, inline: false },
                                { name: `/resume`, value: `Resumes the currently paused track.`, inline: false },
                                { name: `/search`, value: `Search song and play music.`, inline: false },
                                { name: `/seek`, value: `Seeks to a certain point in the current track.`, inline: false },
                                { name: `/shuffle`, value: `Shuffle the queue.`, inline: false },
                                { name: `/skip`, value: `Skip the current song.`, inline: false },
                                { name: `/stop`, value: `Stop the current song and clears the entire music queue.`, inline: false },
                                { name: `/volume`, value: `Changes/Shows the current volume.`, inline: false },
                                )
            */
            commandFiles.map((value) => {
                const file = require(value);
                const splitted = value.split("/");
                const directory = splitted[splitted.length - 2];

                if (file.name) {
                    const properties = { directory, ...file };
                    embed.addFields({ name: `/`, value: ``, inline: false })
                }
            });

            let row = new ActionRowBuilder()
                .addComponents(
                    new ButtonBuilder()
                        .setStyle(ButtonStyle.Link)
                        .setLabel('Invite Bot')
                        .setURL(`https://discord.com/oauth2/authorize?client_id=&permissions=8&scope=bot%20applications.commands`))

                .addComponents(
                    new ButtonBuilder()
                        .setStyle(ButtonStyle.Link)
                        .setLabel('Server Support')
                        .setURL(`https://discord.gg/developer-tools`))

            interaction.reply({ embeds: [embed], components: [row], ephemeral: true })
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder, CommandInteraction } = require('discord.js')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "ping",
    description: "Test the bots response time.",
    async execute(client, interaction) {
        try {
            interaction.reply({ content: `:ping_pong: Pong  ms`, ephemeral: true });
        } catch (err) {
            console.log(err)
        }
    }
}
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { joinVoiceChannel } = require('@discordjs/voice');
const db = require(`quick.db`)
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "247",
    description: "Toggles the 24/7 mode. This makes the bot doesn't leave the voice channel until you stop it.",
    options: [],
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            let channel = interaction.member.voice.channel;
            if (!channel) return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true });

            distube.voices.join(channel).then(() => {
                if (!db.get(`24_7_`)) {
                    db.set(`24_7_`, channel.id)
                    interaction.reply({ content: `:white_check_mark: Successful enabled the 24/7!` });
                } else {
                    db.delete(`24_7_`)
                    interaction.reply({ content: `:white_check_mark: Successful disabled the 24/7!` })
                }
            })
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "autoplay",
    description: "Toggles autoplay for the current guild.",
    async execute(client, interaction) {
        try {
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const mode = distube.toggleAutoplay(interaction)
//            interaction.reply(":white_check_mark: Set autoplay mode to `" + (mode ? "On" : "Off") + "`");
            if (!queue.autoplay) {
                interaction.reply({ content: `:white_check_mark: Set autoplay mode to **` "Off"`**` })
            } else {
                interaction.reply({ content: `:white_check_mark: Set autoplay mode to **` "Off"`**` })
            }
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { joinVoiceChannel } = require('@discordjs/voice');
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "join",
    description: "join the voice channel.",
    options: [],
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId) return interaction.reply({ content: `:no_entry_sign: I'm there already ``` });
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            let channel = interaction.member.voice.channel;
            if (!channel) return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true });
            distube.voices.join(channel).then(() => {
                interaction.reply({ content: `:white_check_mark: Succesfully joined ``` });
            }).catch(() => {
                interaction.reply({ content: `:no_entry_sign: Couldn't join this channel.`, ephemeral: true });
            })
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "jump",
    description: "Skip to a song in the queue.",
    options: [
        {
            name: `number`,
            description: `number of sonds to skip`,
            type: 10,
            required: true
        }
    ],
    async execute(client, interaction) {
        let args = interaction.options.getNumber('number')
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            if (!queue.autoplay && queue.songs.length <= 1) return interaction.reply({ content: `:no_entry_sign:  this is last song in queue list`, ephemeral: true });
            if (0 <= Number(args) && Number(args) <= queue.songs.length) {
                interaction.reply({ content: `:notes: Jumped songs!` })
                return distube.jump(interaction, parseInt(args))
                    .catch(err => interaction.reply({ content: `:no_entry_sign: Invalid song number.`, ephemeral: true }));
            } else {
                interaction.reply({ content: `:no_entry_sign: Please use a number between **0** and ****`, ephemeral: true })
            }
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { joinVoiceChannel } = require('@discordjs/voice');
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "leave",
    description: "leave the voice channel.",
    options: [],
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId !== interaction.member.voice.channelId) return interaction.reply({ content: `:no_entry_sign: I'm not there already ``` });
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            let channel = interaction.member.voice.channel;
            if (!channel) return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true });
            distube.voices.leave(interaction.guild)
            return interaction.reply({ content: `:white_check_mark: Succesfully leave ``` });
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const db = require(`quick.db`)
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "repeat",
    description: "Toggles the repeat mode.",
    options: [
        {
            name: `number`,
            description: `number of sonds to skip`,
            type: 10
        }
    ],
    async execute(client, interaction) {
        let args = interaction.options.getNumber('number')
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            if (0 <= Number(args) && Number(args) <= 2) {
                distube.setRepeatMode(interaction, parseInt(args))
                interaction.reply({ content: `:notes: **Repeat mode set to:** ` })
            } else {
                interaction.reply({ content: `:no_entry_sign: Please use a number between **0** and **2**   |   0: **disabled**, 1: **Repeat a song**, 2: **Repeat all the queue**`, ephemeral: true })
            }
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const { escapeMarkdown } = require("discord.js");
const distube = require('../../client/distube')
const lyricsFinder = require("@jeve/lyrics-finder");
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "lyrics",
    description: "Display lyrics of a song",
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            const song = queue.songs[0]
            let data;
            try {
                data = await lyricsFinder.LyricsFinder(``)
            } catch {
                data = false
            }
            if (!data || !data?.trim) return interaction.reply({ content: `:rolling_eyes: No lyrics found for ****` })
            let embeds0 = [];
            let embeds1 = [];
            let embeds2 = [];
            let embeds3 = [];
            let embeds4 = [];
            let embeds5 = [];
            let embeds6 = [];
            let embeds7 = [];
            let embeds8 = [];
            let embeds9 = [];
            let embeds10 = [];
            if (data.length >= 2048) {
                //const interactions = escapeMarkdown(data, {
                const interactions = splitinteraction(data, {
                    maxLength: 4000,
                    char: '\n',
                });
                for (const interaction of interactions) {
                    let embed = new EmbedBuilder()
                        .setDescription(``)
                    if (!embeds0.length) embed.setTitle(`Lyrics for ""`);
                    if (embeds0.length < 10) { embeds0.push(embed) }
                    else if (embeds0.length >= 10) { embeds1.push(embed) }
                    else if (embeds0.length >= 10 && embeds1.length >= 10) { embeds2.push(embed) }
                    else if (embeds0.length >= 10 && embeds1.length >= 10 && embeds2.length >= 10) { embeds3.push(embed) }
                    else if (embeds0.length >= 10 && embeds1.length >= 10 && embeds2.length >= 10 && embeds3.length >= 10) { embeds4.push(embed) }
                    else if (embeds0.length >= 10 && embeds1.length >= 10 && embeds2.length >= 10 && embeds3.length >= 10 && embeds4.length >= 10) { embeds5.push(embed) }
                    else if (embeds0.length >= 10 && embeds1.length >= 10 && embeds2.length >= 10 && embeds3.length >= 10 && embeds4.length >= 10 && embeds5.length >= 10) { embeds6.push(embed) }
                    else if (embeds0.length >= 10 && embeds1.length >= 10 && embeds2.length >= 10 && embeds3.length >= 10 && embeds4.length >= 10 && embeds5.length >= 10 && embeds6.length >= 10) { embeds7.push(embed) }
                    else if (embeds0.length >= 10 && embeds1.length >= 10 && embeds2.length >= 10 && embeds3.length >= 10 && embeds4.length >= 10 && embeds5.length >= 10 && embeds6.length >= 10 && embeds7.length >= 10) { embeds8.push(embed) }
                    else if (embeds0.length >= 10 && embeds1.length >= 10 && embeds2.length >= 10 && embeds3.length >= 10 && embeds4.length >= 10 && embeds5.length >= 10 && embeds6.length >= 10 && embeds7.length >= 10 && embeds8.length >= 10) { embeds9.push(embed) }
                    else if (embeds0.length >= 10 && embeds1.length >= 10 && embeds2.length >= 10 && embeds3.length >= 10 && embeds4.length >= 10 && embeds5.length >= 10 && embeds6.length >= 10 && embeds7.length >= 10 && embeds8.length >= 10 && embeds9.length >= 10) { embeds10.push(embed) }
                    else if (embeds0.length >= 10 && embeds1.length >= 10 && embeds2.length >= 10 && embeds3.length >= 10 && embeds4.length >= 10 && embeds5.length >= 10 && embeds6.length >= 10 && embeds7.length >= 10 && embeds8.length >= 10 && embeds9.length >= 10 && embeds10.length >= 10) { }
                }
            }
            if (embeds0.length) { interaction.reply({ embeds: embeds0 }); }
            if (embeds1.length) { interaction.reply({ embeds: embeds1 }); }
            if (embeds2.length) { interaction.reply({ embeds: embeds2 }); }
            if (embeds3.length) { interaction.reply({ embeds: embeds3 }); }
            if (embeds4.length) { interaction.reply({ embeds: embeds4 }); }
            if (embeds5.length) { interaction.reply({ embeds: embeds5 }); }
            if (embeds6.length) { interaction.reply({ embeds: embeds6 }); }
            if (embeds7.length) { interaction.reply({ embeds: embeds7 }); }
            if (embeds8.length) { interaction.reply({ embeds: embeds8 }); }
            if (embeds9.length) { interaction.reply({ embeds: embeds9 }); }
            if (embeds10.length) { interaction.reply({ embeds: embeds10 }); }
        } catch (err) {
            console.log(err)
        }
    },
};

function verifyString(
    data,
    error = Error,
    errorinteraction = `Expected a string, got  instead.`,
    allowEmpty = true,
) {
    if (typeof data !== 'string') throw new error(errorinteraction);
    if (!allowEmpty && data.length === 0) throw new error(errorinteraction);
    return data;
}
function splitinteraction(text, { maxLength = 1024, char = '\n', prepend = '', append = '' }) {
    text = verifyString(text);
    //////////////////////////////
    if (text.length <= maxLength) return [text];
    let splitText = [text];
    if (Array.isArray(char)) {
        while (char.length > 0 && splitText.some(elem => elem.length > maxLength)) {
            const currentChar = char.shift();
            if (currentChar instanceof RegExp) {
                splitText = splitText.flatMap(chunk => chunk.match(currentChar));
            } else {
                splitText = splitText.flatMap(chunk => chunk.split(currentChar));
            }
        }
    } else {
        splitText = text.split(char);
    }
    if (splitText.some(elem => elem.length > maxLength)) throw new RangeError('SPLIT_MAX_LEN');
    const interactions = [];
    let msg = '';
    for (const chunk of splitText) {
        if (msg && (msg + char + chunk + append).length > maxLength) {
            interactions.push(msg + append);
            msg = prepend;
        }
        msg += (msg && msg !== prepend ? char : '') + chunk;
    }
    return interactions.concat(msg).filter(m => m);
}
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "nowplaying",
    description: "Shows what is song that the bot is currently playing.",
    options: [],
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            const song = queue.songs[0]
            const uni = ` '➕ | '`;
            const part = Math.floor((queue.currentTime / song.duration) * 30);
            let embed = new EmbedBuilder()
                .setTitle(``)
                .setURL(``)
                .setDescription(`\nCurrent Duration: `[/]`\n${'▇'.repeat(part) + '▇' + '—'.repeat(24 - part)}`)
                .setThumbnail(`https://img.youtube.com/vi//mqdefault.jpg`)
                .setFooter({ text: `@  | Views:  | Like: ` })
            interaction.reply({ embeds: [embed] })
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "pause",
    description: "Pauses the currently playing track.",
    async execute(client, interaction, args) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            const song = queue.songs[0]
            let name = song.name
            if (queue.paused) {
                interaction.reply({ content: `:no_entry_sign: **** has been Paused!`, ephemeral: true })
            } else {
                distube.pause(interaction);
                interaction.reply({ content: `:notes: Paused **** . Type `/resume` to unpause!` })
            }
        } catch (err) {
            console.log(err)
        }
    },
};

const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const wait = require('node:timers/promises').setTimeout;
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "play",
    description: "Add a song to queue and plays it.",
    options: [
        {
            name: `song`,
            description: `the song to play.`,
            type: 3,//string 12
            required: true
        }
    ],
    async execute(client, interaction) {
        let args = interaction.options.getString('song')
        const songTitle = args
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            let player = args//.slice(0).join(' ')
            if (!player) return interaction.reply({ content: `:no_entry_sign: You should type song name or url.`, ephemeral: true })

            const queue = distube.getQueue(interaction)
            interaction.reply({ content: `:watch: Searching ... (``)` })
            await wait(3000);
            await interaction.deleteReply()
            
            const voiceChannel = interaction.member?.voice?.channel;
            if (voiceChannel) {
                distube.play(voiceChannel, player, {
                    interaction,
                    textChannel: interaction.channel,
                    member: interaction.member,
                });
            }
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "previous",
    description: "Plays the previous song in the queue.",
    options: [],
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            if (queue.previousSongs.length == 0) {
                interaction.reply({ content: `:no_entry_sign: There is no previous song in this queue`, ephemeral: true })
            } else {
            await distube.previous(interaction);
            interaction.reply({ content: `:notes: Song has been Previous` })
            }
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder, interactionFlags } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "queue",
    description: "Display the queue of the current tracks in the playlist.",
    options: [],
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })

            /* let curqueue = queue.songs.slice(queue.songs.length / 10).map((song, id) =>
                    `****. [****]() - `
                ).join("\n");*/

            const reload = new ButtonBuilder()
                .setCustomId('reload')
                .setStyle(ButtonStyle.Primary)
                .setLabel('🔄');
            const next = new ButtonBuilder()
                .setCustomId('next')
                .setLabel('➡️')
                .setStyle(ButtonStyle.Primary);
            if (queue.songs.length === 1) next.setDisabled(true)

            const back = new ButtonBuilder()
                .setCustomId('back')
                .setLabel('⬅️')
                .setStyle(ButtonStyle.Primary)
                .setDisabled(true);
            const row = new ActionRowBuilder()
                .addComponents(back, reload, next);

            const exampleEmbed = new EmbedBuilder()
                .setTitle(queue.songs[0].name)
                .setURL(queue.songs[0].url)
                .addFields(
                    { name: 'Time', value: queue.songs[0].formattedDuration, inline: true },
                    { name: 'Views', value: queue.songs[0].views + ' view', inline: true },
                    { name: 'Likes', value: queue.songs[0].likes + ' like', inline: true },
                )
                .setImage(queue.songs[0].thumbnail)
                .setTimestamp()
                .setFooter({ text: `1 / ` });

            return message.reply({
                embeds: [exampleEmbed],
                components: [row]
            })
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "resume",
    description: "Resumes the currently paused track.",
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            const song = queue.songs[0]
            let name = song.name
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            interaction.reply(`:notes: Resumed ****.`)
            return distube.resume(interaction);
        } catch (err) {
            console.log(err)
        }
    },
};
const { ActionRowBuilder, ButtonBuilder, ButtonStyle, Events, EmbedBuilder, StringSelectMenuBuilder } = require("discord.js");
const distube = require('../../client/distube')
const ytsr = require("@distube/ytsr")
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "search",
    description: "Search song and play music.",
    options: [
        {
            name: "song",
            type: 3,
            description: "The song to play.",
            required: true
        }
    ],
    async execute(client, interaction, args) {
        try {
            const string = interaction.options.getString("song");

            await interaction.reply(`:watch: **Searching...** (``)`);

            const message = await interaction.fetchReply();

            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            let channel = interaction.member.voice.channel;
            if (!channel) return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true });

            const row = new ActionRowBuilder()
                .addComponents(
                    new ButtonBuilder()
                        .setCustomId("one")
                        .setEmoji("1️⃣")
                        .setStyle(ButtonStyle.Secondary)
                )
                .addComponents(
                    new ButtonBuilder()
                        .setCustomId("two")
                        .setEmoji("2️⃣")
                        .setStyle(ButtonStyle.Secondary)
                )
                .addComponents(
                    new ButtonBuilder()
                        .setCustomId("three")
                        .setEmoji("3️⃣")
                        .setStyle(ButtonStyle.Secondary)
                )
                .addComponents(
                    new ButtonBuilder()
                        .setCustomId("four")
                        .setEmoji("4️⃣")
                        .setStyle(ButtonStyle.Secondary)
                )
                .addComponents(
                    new ButtonBuilder()
                        .setCustomId("five")
                        .setEmoji("5️⃣")
                        .setStyle(ButtonStyle.Secondary)
                )

            const options = {
                member: interaction.member,
                textChannel: interaction.channel,
                interaction,
            }
            const res = await ytsr(string, { safeSearch: true, limit: 5 });

            let index = 1;
            const result = res.items.slice(0, 5).map(x => `**. []()**`).join("\n")
            const embed = new EmbedBuilder()
                .setAuthor({ name: `Song Selection`, iconURL: interaction.guild.iconURL({ dynamic: true }) })
                .setDescription(result)
                .setFooter({ text: `Please response in 30s` })

            await message.edit({ content: " ", embeds: [embed], components: [row] });
            const filter = i => i.user.id === interaction.user.id;
            const collector = interaction.channel.createMessageComponentCollector({ filter, time: 30000, max: 1 });

            collector.on('collect', async (interaction) => {
                const id = interaction.customId;
                if (id === "one") {
                    await message.delete({ embeds: [], components: [] });
                    await distube.play(interaction.member.voice.channel, res.items[0].url, options);
                } else if (id === "two") {
                    await message.delete({ embeds: [], components: [] });
                    await distube.play(interaction.member.voice.channel, res.items[1].url, options);
                } else if (id === "three") {
                    await message.delete({ embeds: [], components: [] });
                    await distube.play(interaction.member.voice.channel, res.items[2].url, options);
                } else if (id === "four") {
                    await message.delete({ embeds: [], components: [] });
                    await distube.play(interaction.member.voice.channel, res.items[3].url, options);
                } else if (id === "five") {
                    await message.delete({ embeds: [], components: [] });
                    await distube.play(interaction.member.voice.channel, res.items[4].url, options);
                }
            });

            collector.on('end', async (collected, reason) => {
                if (reason === "time") {
                    msg.delete({ content: ``, embeds: [], components: [] });
                }
            });
        } catch (err) {
            console.log(err)
        }
    }
}
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "seek",
    description: "Seeks to a certain point in the current track.",
    options: [
        {
            name: `seconds`,
            description: `the seconds to seek througth the current song.`,
            type: 10,
            required: true
        }
    ],
    async execute(client, interaction) {
        let args = interaction.options.getNumber('seconds')
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            if (!queue.autoplay && queue.formattedCurrentTime <= song.formattedDuration) return message.reply({ content: `:no_entry_sign:  Max formattedDuration: [ / ]`, ephemeral: true });
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!` })
            interaction.reply({ content: `:notes: seeked the song for ` seconds`` })
            return distube.seek(interaction, Number(args * 1000));
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "shuffle",
    description: "Shuffle the queue.",
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            if (!queue.autoplay && queue.songs.length <= 1) return interaction.reply({ content: `:no_entry_sign:  this is last song in queue list` });
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            interaction.reply({ content: `:white_check_mark: Song has been: `Shuffle `` })
            return distube.shuffle(interaction);
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "skip",
    description: "Skip the current song.",
    options: [],
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            const song = queue.songs[0]
            let name = song.name
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            if (!queue.autoplay && queue.songs.length <= 1) return interaction.reply({ content: `:no_entry_sign:  this is last song in queue list`, ephemeral: true });
            interaction.reply({ content: `:notes: Skipped ****` })
            return distube.skip(interaction);
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "stop",
    description: "Stop the current song and clears the entire music queue.",
    options: [],
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            interaction.reply({ content: `:notes: The player has stopped and the queue has been cleared.` })
            return distube.stop(interaction);
        } catch (err) {
            console.log(err)
        }
    },
};
const { EmbedBuilder } = require("discord.js");
const distube = require('../../client/distube')
const db = require(`quick.db`)
const { Utils } = require("devtools-ts");
const utilites = new Utils();

module.exports = {
    name: "volume",
    description: "Changes/Shows the current volume.",
    options: [
        {
            name: `volume`,
            description: `the volume to set`,
            type: 10,
            required: false
        }
    ],
    async execute(client, interaction) {
        try {
            if (interaction.guild.members.me.voice?.channelId && interaction.member.voice.channelId !== interaction.guild.members.me?.voice?.channelId) return interaction.reply({ content: `:no_entry_sign: You must be listening in `` to use that!`, ephemeral: true });
            if (!interaction.member.voice.channel)
                return interaction.reply({ content: ":no_entry_sign: You must join a voice channel to use that!", ephemeral: true })
            const queue = distube.getQueue(interaction)
            if (!queue) return interaction.reply({ content: `:no_entry_sign: There must be music playing to use that!`, ephemeral: true })
            let args = interaction.options.getNumber('volume')
            const volume = parseInt(args);
            if (volume) {
                if (volume < 0 || volume > 150 || isNaN(volume))
                    return interaction.reply({ content: ":no_entry_sign: **Volume must be a valid integer between 0 and 150!**", ephemeral: true })
                if (volume < 0) volume = 0;
                if (volume > 150) volume = 150;
                db.set(`volume_`, volume)
                interaction.reply(`:loud_sound: Volume changed from `` to ```)
                distube.setVolume(interaction, volume);
            } else if (!volume) {
                return interaction.reply({ content: `:loud_sound: Volume: ``%`, ephemeral: true });
            }
        } catch (err) {
            console.log(err)
        }
    },
};

{
    "prefix": "!", "/"، 
    "token": "MTIxNDI2NTUyMzE0ODQzMTQzMQ.GgiF9d.gTmw4k3lHEMLlhPZk3dmyYgmH0-CUCbLCGGkeg",
    "clientID": "1214265523148431431",
    "guildID": "1214232147989373038",
    "youtubeCookie": "",
    "spotify_api": {
        "enabled": true,
        "clientId": "2d2becbcf53a4d778770758c6f9d4a67",
        "clientSecret": "3047e7fbded34ff887a109cd47877338"
    }
}
<div class="header">

<!--Content before waves-->
<div class="inner-header flex">
<!--Just the logo.. Don't mind this-->
  <h1 style="text-align: left;font-family: Aldrich, sans-serif;font-size: 60px;">Developer Tools<br /></h1>

</div>
  
<style>
@import url(//fonts.googleapis.com/css?family=Lato:300:400);

body {
  margin:0;
}

h1 {
  font-family: 'Lato', sans-serif;
  font-weight:300;
  letter-spacing: 2px;
  font-size:48px;
}
p {
  font-family: 'Lato', sans-serif;
  letter-spacing: 1px;
  font-size:14px;
  color: #333333;
}

.header {
  position:relative;
  text-align:center;
  background: linear-gradient(60deg, rgba(251,79,33,1) 0%, rgba(242,35,87,1) 100%);
  color:white;
}
.logo {
  width:50px;
  fill:white;
  padding-right:15px;
  display:inline-block;
  vertical-align: middle;
}

.inner-header {
  height:65vh;
  width:100%;
  margin: 0;
  padding: 0;
}

.flex { /*Flexbox for containers*/
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.waves {
  position:relative;
  width: 100%;
  height:15vh;
  margin-bottom:-7px; /*Fix for safari gap*/
  min-height:100px;
  max-height:150px;
}

.content {
  position:relative;
  height:20vh;
  text-align:center;
  background-color: white;
}

/* Animation */

.parallax > use {
  animation: move-forever 25s cubic-bezier(.55,.5,.45,.5)     infinite;
}
.parallax > use:nth-child(1) {
  animation-delay: -2s;
  animation-duration: 7s;
}
.parallax > use:nth-child(2) {
  animation-delay: -3s;
  animation-duration: 10s;
}
.parallax > use:nth-child(3) {
  animation-delay: -4s;
  animation-duration: 13s;
}
.parallax > use:nth-child(4) {
  animation-delay: -5s;
  animation-duration: 20s;
}
@keyframes move-forever {
  0% {
   transform: translate3d(-90px,0,0);
  }
  100% { 
    transform: translate3d(85px,0,0);
  }
}
/*Shrinking for mobile*/
@media (max-width: 768px) {
  .waves {
    height:40px;
    min-height:40px;
  }
  .content {
    height:30vh;
  }
  h1 {
    font-size:24px;
  }
}
</style>

<!--Waves Container-->
<div>
<svg class="waves" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"
viewBox="0 24 150 28" preserveAspectRatio="none" shape-rendering="auto">
<defs>
<path id="gentle-wave" d="M-160 44c30 0 58-18 88-18s 58 18 88 18 58-18 88-18 58 18 88 18 v44h-352z" />
</defs>
<g class="parallax">
<use xlink:href="#gentle-wave" x="48" y="0" fill="rgba(255,255,255,0.7" />
<use xlink:href="#gentle-wave" x="48" y="3" fill="rgba(255,255,255,0.5)" />
<use xlink:href="#gentle-wave" x="48" y="5" fill="rgba(255,255,255,0.3)" />
<use xlink:href="#gentle-wave" x="48" y="7" fill="#fff" />
</g>
</svg>
</div>
<!--Waves end-->

</div>
<!--Header ends-->

<!--Content starts-->
<div class="content flex">
<p>Copyright © 2023 Developer-tools All rights reserved
</p>
</div>
<!--Content ends-->
SQLite format 3   @                                                                     ._
   � �                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      :YtablejsonjsonCREATE TABLE json (ID TEXT, json TEXT)
                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              
# AstroMusic : Distube Bot
Listening to High-Quality Music from Various Singers

### Created by Developer Tools <img style="width: 5%; float: left; margin-right: 20px;" src="https://media.discordapp.net/attachments/965806495109369926/1109147334199214171/png_2dev_copy.png?width=409&height=409" alt="AstroMusic Illustration"> 


## Supports Slash Commands & Prefix


- SoundCloud
- YouTube
- Deezer
- Spotify

This Discord bot allows you to control music playback using either slash commands or a designated prefix.

## About AstroMusic

AstroMusic is a versatile Discord bot designed for music enthusiasts. It enables users to listen to music from various platforms in high quality. The bot supports both slash commands and traditional prefixes for controlling its functions.

## Supported Platforms

### SoundCloud
Enjoy tracks from SoundCloud's extensive collection. Simply use the bot's commands to search for and play your favorite SoundCloud songs.

### YouTube
Explore an endless library of music on YouTube using AstroMusic. The bot lets you easily search for and stream YouTube videos without leaving your Discord server.

### Deezer
With AstroMusic, you can access Deezer's diverse music catalog. Stream tracks seamlessly and enhance your listening experience.

### Spotify
AstroMusic provides access to Spotify's vast music library. Listen to your favorite Spotify songs and playlists right within your Discord server.

## How to Use

AstroMusic is designed to be user-friendly. You can control the bot's functions using either slash commands or a prefix of your choice. This flexibility ensures that you can interact with the bot in a way that suits your preferences.

To start playing music, simply use the provided commands to search for tracks from the supported platforms. You can also control playback, skip tracks, adjust volume, and perform various other actions using the bot's intuitive commands.

## Getting Started

To add AstroMusic to your Discord server and start enjoying your favorite tunes, follow these steps:

1. Invite the bot to your server using the provided invite link.
2. Grant the necessary permissions for the bot to join voice channels and interact in text channels.
3. Begin using the bot by invoking its commands through slash commands or the designated prefix.



Here's an image that provides an illustration of AstroMusic in action:

<img style="width: 50%;" src="https://media.discordapp.net/attachments/1113676076732911761/1144333970432086026/Screen_Shot_2023-08-23_at_1.png?width=416&height=292" alt="AstroMusic Illustration">

## Conclusion

AstroMusic brings the joy of music to your Discord server. With its support for multiple platforms and user-friendly controls, you and your server members can immerse yourselves in the world of music without any hassle.

For detailed information about the available commands and how to make the most of AstroMusic, refer to the bot's documentation and help resources.

Feel the rhythm with AstroMusic today!

{
  "name": "distube",
  "version": "1.6.7",
  "description": "distube: Listening music singers in high quality get support quickly from: https://discord.gg/devtools-931536214228611102",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "@discordjs/builders": "^1.4.0",
    "@discordjs/opus": "^0.8.0",
    "@discordjs/rest": "^1.5.0",
    "@discordjs/voice": "^0.16.0",
    "@distube/deezer": "^1.0.0",
    "@distube/soundcloud": "^1.3.0",
    "@distube/spotify": "^1.5.1",
    "@distube/yt-dlp": "^1.1.3",
    "@distube/ytdl-core": "^4.13.1",
    "@distube/ytsr": "^1.1.9",
    "@jeve/lyrics-finder": "^1.0.1",
    "ascii-table": "^0.0.9",
    "axios": "^1.2.2",
    "colors": "^1.4.0",
    "delay": "^5.0.0",
    "devtools-ts": "^1.1.3",
    "discord-api-types": "^0.37.27",
    "discord-player": "^5.1.0",
    "discord.js": "^14.8.0",
    "distube": "^4.0.4",
    "events": "^3.3.0",
    "express": "^4.18.2",
    "ffmpeg-static": "^5.1.0",
    "glob": "^8.0.3",
    "ms": "^2.1.3",
    "node-fetch": "^3.3.0",
    "opusscript": "^0.0.8",
    "quick.db": "^7.1.3",
    "spotify-url-info": "^3.2.3",
    "string-progressbar": "^1.0.4",
    "winston": "^3.8.2",
    "ytdl-core": "^4.11.2"
  },
  "repository": {
    "type": "git",
    "url": "https://github.com/Developer-Tools-Discord/AstroMusic"
  },
  "keywords": [],
  "devDependencies": {}
}

