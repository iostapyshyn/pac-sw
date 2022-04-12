TARGET_GCC = $(CROSS_COMPILE)$(CC)

PLUGIN = pac_sw_plugin.so
SRC = main.cc

PROLOGUE ?= asm/prologue.S
EPILOGUE ?= asm/epilogue.S

PROLOGUE_IN = prologue.s
EPILOGUE_IN = epilogue.s

TEST = test
MAP_DEVICE = map_device.o

# Plugins directory of GCC
GCC_PLUGINS_DIR = $(shell $(TARGET_GCC) -print-file-name=plugin)

# GCC CPP flags (we need pic and no-rtti)
CXXFLAGS += -g -I$(GCC_PLUGINS_DIR)/include -fPIC -fno-rtti -DDEBUG

.PHONY: all clean

all: $(PLUGIN)

# Preprocess assembler files before embedding
$(PROLOGUE_IN): $(PROLOGUE)
$(EPILOGUE_IN): $(EPILOGUE)

$(PROLOGUE_IN) $(EPILOGUE_IN):
	$(TARGET_GCC) -E $^ > $@

$(PLUGIN): $(SRC)
	$(CXX) -shared $(CXXFLAGS) $^ -o $@


$(TEST): $(TEST).c $(MAP_DEVICE) $(PLUGIN) $(PROLOGUE_IN) $(EPILOGUE_IN)
	$(TARGET_GCC) $< $(MAP_DEVICE) \
		-fplugin=./$(PLUGIN) \
		-fplugin-arg-pac_sw_plugin-prologue=$(PROLOGUE_IN) \
		-fplugin-arg-pac_sw_plugin-epilogue=$(EPILOGUE_IN) \
		-fplugin-arg-pac_sw_plugin-init-function=map_device \
		-g -O0 -o $@

# testnostd: gcc_pac_plugin.so
# 	$(TARGET_GCC) -E prologue.S > prologue.S.proc
# 	$(TARGET_GCC) -E epilogue.S > epilogue.S.proc
# 	$(TARGET_GCC) mpeg2.c memmapper.c \
# 	-fplugin=$(shell pwd)/gcc_pac_plugin.so \
# 	-fplugin-arg-gcc_pac_plugin-needpa=y \
# 	-fplugin-arg-gcc_pac_plugin-memmap=y \
# 	-fplugin-arg-gcc_pac_plugin-prologue=prologue.S.proc \
# 	-fplugin-arg-gcc_pac_plugin-epilogue=epilogue.S.proc \
# 	-g -O0 -o bin

clean:
	rm -fv $(PLUGIN) $(TEST) $(PROLOGUE_IN) $(EPILOGUE_IN)