#
# Makaqu Makefile for Sega Dreamcast (KallistiOS)
# Compiled with a KOS 1.3.x snapshot
# June 2010
#

TARGET = Makaqu.elf
BIN = Makaqu.bin
FIRSTREAD = 1st_read.bin
KOS_LOCAL_CFLAGS = -I../ -Iinclude -Os
LIBS=-lm -lz -lSDL
# -lSDL
# -lSDL_mixer -lSDL_audio

CDRECORD_ID = ATAPI:0,0,0
CDRECORD_DEV = /dev/hdc

# Common source files for all versions
COMMON_OBJS = chase.o cl_demo.o cl_input.o cl_main.o cl_parse.o cl_tent.o cmd.o
COMMON_OBJS += common.o console.o crc.o cvar.o host.o host_cmd.o
COMMON_OBJS += keys.o mathlib.o menu.o net_loop.o net_main.o net_vcr.o
COMMON_OBJS += nonintel.o pr_cmds.o pr_edict.o pr_exec.o sbar.o
COMMON_OBJS += sv_main.o sv_move.o sv_phys.o sv_user.o view.o wad.o
COMMON_OBJS += world.o zone.o

# Uncomment this when we have a working sound driver
COMMON_OBJS += snd_dma.o snd_mem.o snd_mix.o

# Software rendering version
SOFT_OBJS = d_edge.o d_fill.o d_init.o d_modech.o d_scan.o
SOFT_OBJS += r_part.o d_part.o
SOFT_OBJS += spr_file.o spr_r.o spr_d.o
SOFT_OBJS += mdl_file.o mdl_rend.o mdl_clip.o mdl_draw.o
SOFT_OBJS += d_sky.o d_surf.o d_vars.o d_zpoint.o draw.o
SOFT_OBJS += r_bsp.o r_draw.o r_edge.o r_efrag.o r_light.o
SOFT_OBJS += r_sky.o r_surf.o r_vars.o model.o
SOFT_OBJS += tex_file.o r_misc.o r_2d.o croshair.o palette.o screen.o r_main.o

# System specific stuff
SYS_OBJS = cd_dc.o dc_menu.o in_dc.o net_dc.o sys_dc.o vid_dc.o snd_sdl.o
# snd_dc.o

SYS_OBJS += dc_vmu.o dc_vibe.o
SYS_OBJS += kos_zlib.o

OBJS = $(COMMON_OBJS) $(SOFT_OBJS) $(SYS_OBJS)

all: rm-elf $(TARGET)

include $(KOS_BASE)/Makefile.rules

clean:
	-rm -f $(TARGET) $(BIN) $(FIRSTREAD) $(OBJS)

rm-elf:
	-rm -f $(TARGET)

$(TARGET): $(OBJS)
	$(KOS_CC) $(KOS_CFLAGS) $(KOS_LDFLAGS) -o $(TARGET) $(KOS_START) \
		$(OBJS) $(OBJEXTRA) -L$(KOS_BASE)/lib $(LIBS) -lm $(KOS_LIBS)

%.o: src/%.c
	$(KOS_CC) $(KOS_CFLAGS) $(KOS_LOCAL_CFLAGS) $(OPT_FLAGS) -c $< -o $@

%.o: ../%.c
	$(KOS_CC) $(KOS_CFLAGS) $(KOS_LOCAL_CFLAGS) $(OPT_FLAGS) -c $< -o $@

%.o: ../%.c
	$(KOS_CC) $(KOS_CFLAGS) $(KOS_LOCAL_CFLAGS) $(OPT_FLAGS) -c $< -o $@

%.o: ../%.c
	$(KOS_CC) $(KOS_CFLAGS) $(KOS_LOCAL_CFLAGS) $(OPT_FLAGS) -c $< -o $@

# Run over serial or network connection
run: $(TARGET)
	$(KOS_LOADER) $(TARGET)

# Create unscrambled binary
$(BIN): $(TARGET)
#	$(KOS_STRIP) $(TARGET)
	$(KOS_OBJCOPY) -R .stack -O binary $(TARGET) $(BIN)

# Create scrambled binary
$(FIRSTREAD): $(BIN)
	$(KOS_BASE)/utils/scramble/scramble $(BIN) $(FIRSTREAD)
#	$(KOS_CC_BASE)/bin/sh-elf-scramble $(BIN) $(FIRSTREAD)

# Start a new multi-session CD (unscrambled binary)
newcd: $(BIN)
	mv $(BIN) disc/data/1st_read.bin
	mkisofs -l -J -R -o disc/session1.iso disc/data
	rm disc/data/1st_read.bin
	cdrecord dev=$(CDRECORD_ID) gracetime=2 -multi -eject -xa disc/session1.iso
	rm disc/session?.iso

# Continue a multi-session CD (unscrambled binary)
mscd: $(BIN)
	mv $(BIN) disc/data/1st_read.bin
	mkisofs -C `cdrecord dev=$(CDRECORD_ID) -msinfo` -M $(CDRECORD_DEV) -l -J -R -o disc/session2.iso disc/data
	rm disc/data/1st_read.bin
	cdrecord dev=$(CDRECORD_ID) gracetime=2 -multi -eject -xa disc/session2.iso
	rm disc/session?.iso

# Burn a bootable CD
selfboot: $(FIRSTREAD)
	mkisofs -G disc/IP.BIN -l -J -R -o disc/session1.iso disc/data $(FIRSTREAD)
	dd if=disc/session1.iso bs=2048 count=17 of=disc/session2.iso
	dd if=/dev/zero bs=2048 count=283 >> disc/session2.iso
	cdrecord dev=$(CDRECORD_ID) -multi -xa disc/session1.iso
	cdrecord dev=$(CDRECORD_ID) -eject -xa disc/session2.iso
	rm disc/session?.iso
