TARGET=avr-atmega128rfa1
MCU=atmega128rfa1
PCB = LIGHT_SW_PCB_R1=1
#PCB = BLIND_PCB_R0=1
#PCB = XBEE_PCB_R1=1

all: er-example-server 
#er-example-server 
#er-example-client
# Use this target explicitly if requried: er-plugtest-server

OUTFILE=er-example-server.$(TARGET)
#OUTFILE=er-example-client.$(TARGET)

#	${MAKE} TARGET=$(TARGET) DEFINES=$(PCB) NOAVRSIZE=1 er-example-server.elf   
#can't just rename, webserver6 may be open in debugger

CONTIKI=../..
CFLAGS += -DPROJECT_CONF_H=\"project-conf.h\"

# for static routing, if enabled
ifneq ($(TARGET), minimal-net)
ifneq ($(TARGET), native)
ifneq ($(findstring avr,$(TARGET)), avr)
PROJECT_SOURCEFILES += static-routing.c
endif
endif
endif

# variable for root Makefile.include
WITH_UIP6=1
# for some platforms
UIP_CONF_IPV6=1

# variable for this Makefile
# configure CoAP implementation (3|7) (er-coap-07 also supports CoAP draft 08)
WITH_COAP=7

# new variable since slip-radio
ifneq ($(TARGET), minimal-net)
UIP_CONF_RPL=0
else
# minimal-net does not support RPL under Linux and is mostly used to test CoAP only
${info INFO: compiling without RPL}
UIP_CONF_RPL=0
CFLAGS += -DUIP_CONF_ND6_DEF_MAXDADNS=0
CFLAGS += -DHARD_CODED_ADDRESS=\"fdfd::10\"
CFLAGS += -DUIP_CONF_BUFFER_SIZE=1280
endif

# linker optimizations
SMALL=1

# REST framework, requires WITH_COAP
ifeq ($(WITH_COAP), 7)
${info INFO: compiling with CoAP-08}
CFLAGS += -DWITH_COAP=7
CFLAGS += -DREST=coap_rest_implementation
CFLAGS += -DUIP_CONF_TCP=0
APPS += er-coap-07
else ifeq ($(WITH_COAP), 3)
${info INFO: compiling with CoAP-03}
CFLAGS += -DWITH_COAP=3
CFLAGS += -DREST=coap_rest_implementation
CFLAGS += -DUIP_CONF_TCP=0
APPS += er-coap-03
else
${info INFO: compiling with HTTP}
CFLAGS += -DWITH_HTTP
CFLAGS += -DREST=http_rest_implementation
CFLAGS += -DUIP_CONF_TCP=1
APPS += er-http-engine
endif

APPS += erbium

# optional rules to get assembly
#CUSTOM_RULE_C_TO_OBJECTDIR_O = 1
#CUSTOM_RULE_S_TO_OBJECTDIR_O = 1

include $(CONTIKI)/Makefile.include

# optional rules to get assembly
#$(OBJECTDIR)/%.o: asmdir/%.S
#	$(CC) $(CFLAGS) -MMD -c $< -o $@
#	@$(FINALIZE_DEPENDENCY)
#
#asmdir/%.S: %.c
#	$(CC) $(CFLAGS) -MMD -S $< -o $@

# border router rules
$(CONTIKI)/tools/tunslip6:	$(CONTIKI)/tools/tunslip6.c
	(cd $(CONTIKI)/tools && $(MAKE) tunslip6)

connect-router:	$(CONTIKI)/tools/tunslip6
	sudo $(CONTIKI)/tools/tunslip6 aaaa::1/64

connect-router-cooja:	$(CONTIKI)/tools/tunslip6
	sudo $(CONTIKI)/tools/tunslip6 -a 127.0.0.1 aaaa::1/64

tap0up:
	sudo ip address add fdfd::1/64 dev tap0
	
all:
	avr-objcopy -O ihex -R .eeprom -R .fuse -R .signature $(OUTFILE) $(OUTFILE).hex
	avr-objcopy -O ihex -j .eeprom --set-section-flags=.eeprom="alloc,load" --change-section-lma .eeprom=0 $(OUTFILE) $(OUTFILE)_eeprom.hex
	avr-size -C --mcu=$(MCU) $(OUTFILE)

clean:
	rm $(OUTFILE)
	rm -rf obj_avr-atmega128rfa1
	rm *.hex *.map *.a
