# pull content from dalvik/linux device keeping perms/owners
# Copyright (C) 2012  alexander barakin aka sash-kan <alex@barak.in>
#
# This program is free software: you can redistribute it and/or modify
# it under the terms of the GNU General Public License as published by
# the Free Software Foundation, either version 3 of the License, or
# (at your option) any later version.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program.  If not, see <http://www.gnu.org/licenses/>.

s:=$(shell which bash)
ifneq ($(s),)
	SHELL=$(s)
endif

d=system
# make proper dir name: system -> /system
dir=/$(patsubst /%,%,$(patsubst %/,%,$(d)))
parent=$(dir $(dir))
name=$(notdir $(dir))
tar=$(name).tgz
image=$(name).img
tmp:=$(shell mktemp -du)
date:=$(shell date +%F.%T)
log:=$(name).log.$(date)
# default permission and owner
perm=u=rwx,g=rx,o=rx
owner=root:root

adb=$(shell which adb)
mkyaffs=$(shell which mkyaffs2image)

ifneq ($(adb),)
device=$(shell $(adb) devices | sed -rn 's/^(\w+)\s+device.*/\1/p')
endif
ifneq ($(device),)
dev=-s $(device)
endif

ifeq ($(MAKECMDGOALS),)
targets=dotar
else
targets=$(patsubst %,do%,$(filter tar image all,$(MAKECMDGOALS)))
endif

# default target (really - tar)
default: common

all: common

tar: common

image: common

common: checkshell checkadb checkfakeroot pull $(targets) cleantmp

dotar: cleantar $(tar)

cleantar:
	-rm -f $(tar)

doimage: cleanimage $(image)

cleanimage:
	-rm -f $(image)

doall: dotar doimage

pull: $(tmp)/$(name)

$(tmp)/%:
	@echo pull from $(parent)$*/
	@mkdir -p $@; chown $(owner) $@; chmod $(perm) $@;
	@declare -A user user[adb]=1011; user[audio]=1005; user[bluetooth]=1002; \
	user[cache]=2001; user[camera]=1006; user[compass]=1008; user[dhcp]=1014; \
	user[diag]=2002; user[gps]=1021; user[graphics]=1003; user[inet]=3003; \
	user[input]=1004; user[install]=1012; user[keystore]=1017; user[log]=1007; \
	user[media]=1013; user[misc]=9998; user[mount]=1009; user[net_admin]=3005; \
	user[net_bt]=3002; user[net_bt_admin]=3001; user[net_raw]=3004; user[nfc]=1025; \
	user[nobody]=9999; user[radio]=1001; user[root]=0; user[sdcard_rw]=1015; \
	user[shell]=2000; user[system]=1000; user[usb]=1018; user[vpn]=1016; user[wifi]=1010; \
	$(adb) $(dev) shell ls -a -l $(parent)$*/ | \
	while read str; do \
		echo $$str | grep -q "Not a directory" && { echo $$str >> $(log); rmdir $@; break; }; \
		echo $$str | grep -q "Permission denied" && { echo "$(parent)$*/: $$str" >> $(log); break; }; \
		echo $$str | sed -r 's/\r//g;s/^(..........) +(\w+) +(\w+) +([0-9]+ )?....-..-.. ..:.. ([^ ]+)( -> ([^ ]+))?/ \1 \2 \3 \5 \7/;s/^ (.)(...)(...)(...) / \1 \2 \3 \4 /' | \
		while read type pu pg po uu ug nm tg; do \
			owner=$${user[$$uu]}:$${user[$$ug]}; perm=$$(echo "u=$$pu,g=$$pg,o=$$po" | sed 's/-//g'); \
			thename=$$(echo "$(parent)$(*)/$$nm" | sed 's/\r//g'); \
			case $$type in \
				-) res=$$($(adb) $(dev) pull $$thename $@ 2>&1); \
					echo $$res | grep -q "Permission denied" && { echo "$$thename: $$res" >> $(log); continue; }; \
					chown $$owner $@/$$nm; chmod $$perm $@/$$nm \
				;; \
				l) ln -sf $$tg $@/$$nm \
				;; \
				d) $(MAKE) --no-print-directory tmp=$(tmp) log=$(log) $@/$$nm owner=$$owner perm=$$perm \
				;; \
				*) echo "$$thename: unknown type" >> $(log) \
				;; \
			esac; \
		done; \
	done

$(tar):
	@tar -czf $@ -C $(tmp) $(name)

$(image):
	@$(mkyaffs) $(tmp)/$(name) $@

cleantmp:
ifneq ($(tmp),)
	-rm -rf $(tmp)
endif

checkshell:
ifeq ($(s),)
	$(info we need bash!)
	$(error no bash found)
endif

checkfakeroot:
ifeq ($(FAKEROOTKEY),)
	$(info for help run: "make help")
	$(error no fakeroot)
endif

checkadb:
ifeq ($(adb),)
	$(info for help run: "make help")
	$(error no adb specified)
else
ifeq ($(words $(device)),0)
	$(info for help run: "make help")
	$(error device is not specified and not obtained from adb output)
else
ifneq ($(words $(device)),1)
	$(info for help run: "make help")
	$(info adb detects more than one device ($(device)))
	$(error please specify one device with "device=...")
else
ifeq ($(shell $(adb) devices | grep $(device)),)
	$(info for help run: "make help")
	$(error no device with serial $(device) found by adb)
endif
endif
endif
endif

.PHONY: help default tar dotar image doimage common
.PHONY: checkshell checkadb checkfakeroot
.PHONY: cleantmp pull all doall
.PHONY: cleantar cleanimage

help:
	$(info )
	$(info usage: fakeroot make [targets] [variables assignement] ...)
	$(info )
	$(info supported targets:)
	$(info help  # this screen)
	$(info tar   # make tarboll; needs only fakeroot and adb)
	$(info image # make image and tarboll; needs also mkyaffs2image)
	$(info )
	$(info fakeroot can be installed from your distro repository)
	$(info )
	$(info this script also needs "adb" from dalvik/linux sdk)
	$(info )
	$(info optionally (for building target "image"))
	$(info this script also needs "mkyaffs2image" from dalvik/linux sdk)
	$(info )
	$(info to obtain and setup dalvik/linux sdk, follow one of these links:)
	$(info 1. http://developer.android.com/sdk/)
	$(info smaller downloads, but contains blobs)
	$(info 2. http://source.android.com/source/initializing.html)
	$(info much larger downloads, but only free software)
	$(info )
	$(info hints about sdk stuffs:)
	$(info follow above instruction and after running "lunch ..." just)
	$(info run "make adb mkyaffs2image" to build adb and mkyaffs2image)
	$(info )
	$(info if you don't wont/can change your $$PATH,)
	$(info  you can specify paths to abf and/or mkyaffs2image:)
	$(info fakeroot make adb=/full/path/to/adb mkyaffs=/full/path/to/mkyaffs2image ...)
	$(info )
	$(info if you have more than one device connected (see output of "adb devices"))
	$(info you should specify your device serial with "device=...":)
	$(info fakeroot make device=your_device_serial ...)
	$(info )

