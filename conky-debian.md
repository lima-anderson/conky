# Configuração do Conky para Autostart usando systemd de Usuário no Debian 13

## 1. Problema

- Rodar Conky manualmente (`/bin/conky -d`) funciona.
- Adicionar Conky no `~/.config/autostart` com `Exec=/bin/conky -d` falha, porque o GNOME inicia o Conky **antes do desktop estar totalmente carregado**, causando falha silenciosa.
- Tentar editar `~/.config/systemd/user/conky.service` com `sudo` cria arquivos com permissões de root, que o **systemd de usuário não consegue usar**.

---

## 2. Preparar o ambiente

1. **Criar a pasta do systemd de usuário (se não existir)**

```bash
mkdir -p ~/.config/systemd/user
```

2. **Verificar propriedade da pasta**

```bash
ls -ld ~/.config/systemd/user
```

- Se aparecer `root root`, o Conky não vai conseguir iniciar.

3. **Corrigir propriedade**

```bash
sudo chown -R anders:anders ~/.config/systemd/user
```

- Substitua `anders` pelo seu usuário.

---

## 3. Criar o serviço systemd para Conky

1. **Abrir o arquivo com vim (sem sudo!)**

```bash
vim ~/.config/systemd/user/conky.service
```

2. **Inserir conteúdo do serviço**

No vim, pressione `i` e cole:

```ini
[Unit]
Description=Conky Desktop Widget
After=graphical.target

[Service]
Type=simple
ExecStart=/bin/conky -d
Restart=always
Environment=DISPLAY=:0
Environment=XAUTHORITY=/home/anders/.Xauthority

[Install]
WantedBy=default.target
```

3. **Salvar e sair do vim**

- Pressione `Esc`
- Digite `:wq` e pressione Enter

---

## 4. Recarregar systemd de usuário

```bash
systemctl --user daemon-reload
```

---

## 5. Habilitar o serviço para iniciar automaticamente no login

```bash
systemctl --user enable conky.service
```

---

## 6. Testar o serviço imediatamente

```bash
systemctl --user start conky.service
systemctl --user status conky.service
```

- Se o Conky aparecer na tela, a configuração está funcionando.
- A partir do próximo login, o Conky iniciará sozinho sem necessidade de scripts com `sleep`.

---

## 7. Observações importantes

- **Nunca use sudo** ao criar ou editar arquivos em `~/.config/systemd/user/`.
- O **systemd de usuário funciona apenas para o usuário atual**. Serviços criados como root não são visíveis para o `--user`.
- O **GNOME/Wayland** moderno pode dificultar que Conky apareça no login; esta configuração funciona melhor em **GNOME on Xorg**.
- O arquivo `conky.conf` pode ser configurado normalmente; `own_window_type = 'override'` e `own_window_transparent = true` funcionam bem para que o Conky fique visível sobre o papel de parede.

___________________


```lua
conky.config = {
	
	update_interval = 1,
	cpu_avg_samples = 2,
	net_avg_samples = 2,
	out_to_console = false,
	override_utf8_locale = true,
	double_buffer = true,
	no_buffers = true,
	text_buffer_size = 32768,
	imlib_cache_size = 0,
	own_window = true,
	own_window_type = 'desktop',
	own_window_transparent = true,
	own_window_argb_visual = true,
	own_window_argb_value = 50,
	own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager',
	border_inner_margin = 5,
	border_outer_margin = 0,
	xinerama_head = 1,
	alignment = 'bottom_right',
	gap_x = 0,
	gap_y = 33,
	draw_shades = false,
	draw_outline = false,
	draw_borders = false,
	draw_graph_borders = false,
	use_xft = true,
	font = 'Ubuntu Mono:size=12',
	xftalpha = 0.8,
	uppercase = false,
	default_color = 'white',
	own_window_colour = '#000000',
	minimum_width = 300, minimum_height = 0,
	alignment = 'top_right',

};
conky.text = [[
${time %H:%M:%S}${alignr}${time %d-%m-%y}
${voffset -16}${font sans-serif:bold:size=18}${alignc}${time %H:%M}${font}
${voffset 4}${alignc}${time %A, %d de %B de %Y}
${font}${voffset -4}
${font sans-serif:bold:size=10}SYSTEM ${hr 2}
${font sans-serif:normal:size=8}$sysname $kernel $alignr $machine
Host:$alignr$nodename
Uptime:$alignr$uptime
File System: $alignr${fs_type}
Processes: $alignr ${execi 1000 ps aux | wc -l}

${font sans-serif:bold:size=10}CPU ${hr 2}
${font sans-serif:normal:size=8}${execi 1000 grep model /proc/cpuinfo | cut -d : -f2 | tail -1 | sed 's/\s//'}
${font sans-serif:normal:size=8}${cpugraph cpu1}
CPU: ${cpu cpu1}% ${cpubar cpu1}

${font sans-serif:bold:size=10}MEMORY ${hr 2}
${font sans-serif:normal:size=8}RAM $alignc $mem / $memmax $alignr $memperc%
$membar
SWAP $alignc ${swap} / ${swapmax} $alignr ${swapperc}%
${swapbar}

${font sans-serif:bold:size=10}DISK USAGE ${hr 2}
${font sans-serif:normal:size=8}/home $alignc ${fs_used /home} / ${fs_size /home} $alignr ${fs_used_perc /home}%
${fs_bar /home}

${font Ubuntu:bold:size=10}NETWORK ${hr 2}
${if_existing /sys/class/net/wlo1/}
Wi-Fi
Down: ${downspeed wlo1} ${alignr}Up: ${upspeed wlo1}
${color lightgray}${downspeedgraph wlo1 80,110 } ${alignr}${upspeedgraph wlo1 80,130 }$color
${else}
${endif}
${font sans-serif:bold:size=10}TOP PROCESSES ${hr 2}
${font sans-serif:normal:size=8}NAME $alignr PID   CPU%   MEM%${font sans-serif:normal:size=8}
${top name 1} $alignr ${top pid 1} ${top cpu 1}% ${top mem 1}%
${top name 2} $alignr ${top pid 2} ${top cpu 2}% ${top mem 2}%
${top name 3} $alignr ${top pid 3} ${top cpu 3}% ${top mem 3}%
${top name 4} $alignr ${top pid 4} ${top cpu 4}% ${top mem 4}%
${top name 5} $alignr ${top pid 5} ${top cpu 5}% ${top mem 5}%
${top name 6} $alignr ${top pid 6} ${top cpu 6}% ${top mem 6}%
${top name 7} $alignr ${top pid 7} ${top cpu 7}% ${top mem 7}%
${top name 8} $alignr ${top pid 8} ${top cpu 8}% ${top mem 8}%
${top name 9} $alignr ${top pid 9} ${top cpu 9}% ${top mem 9}%
${top name 10} $alignr ${top pid 10} ${top cpu 10}% ${top mem 10}%
]];

```
