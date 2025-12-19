# 检测操作系统和架构
UNAME_S := $(shell uname -s)
UNAME_M := $(shell uname -m)

# 默认值
CXX = g++
CXXFLAGS = -I../common -Iobjs/ -O3 -Wall
ISPC = ispc
TASKSYS_LIB = -lpthread

# 根据操作系统和架构设置不同的标志
ifeq ($(UNAME_S),Darwin)
    # macOS系统
    ifeq ($(UNAME_M),arm64)
        # Apple Silicon (M系列芯片)
        ISPCFLAGS = -O2 --target=neon-i32x4 --arch=aarch64
        CXXFLAGS += -arch arm64
    else ifeq ($(UNAME_M),x86_64)
        # Intel Mac
        ISPCFLAGS = -O2 --target=avx2-i32x8 --arch=x86-64
        CXXFLAGS += -arch x86_64
    endif
else ifeq ($(UNAME_S),Linux)
    # Linux系统
    ifeq ($(UNAME_M),x86_64)
        # x86_64 Linux
        CXXFLAGS += -m64
        # 检测是否支持AVX2（可选）
        ifneq ($(shell grep avx2 /proc/cpuinfo 2>/dev/null),)
            ISPCFLAGS = -O2 --target=avx2-i32x8 --arch=x86-64
        else
            # 降级到SSE4或AVX
            ISPCFLAGS = -O2 --target=sse4-i32x8 --arch=x86-64
        endif
    else ifeq ($(UNAME_M),aarch64)
        # ARM64 Linux
        ISPCFLAGS = -O2 --target=neon-i32x4 --arch=aarch64
    else
        # 其他架构
        $(warning 未知架构: $(UNAME_M)，使用默认设置)
        ISPCFLAGS = -O2 --target=generic
    endif
else
    # 其他操作系统
    $(warning 未知操作系统: $(UNAME_S)，使用默认设置)
    ISPCFLAGS = -O2 --target=generic
endif

# 应用程序名称
APP_NAME = sqrt

# 目录设置
OBJDIR = objs
COMMONDIR = ../common

# PPM文件处理
PPM_CXX = $(COMMONDIR)/ppm.cpp
PPM_OBJ = $(addprefix $(OBJDIR)/, $(subst $(COMMONDIR)/,, $(PPM_CXX:.cpp=.o)))

# 任务系统文件处理
TASKSYS_CXX = $(COMMONDIR)/tasksys.cpp
TASKSYS_OBJ = $(addprefix $(OBJDIR)/, $(subst $(COMMONDIR)/,, $(TASKSYS_CXX:.cpp=.o)))

# 所有对象文件
OBJS = $(OBJDIR)/main.o \
       $(OBJDIR)/sqrtSerial.o \
       $(OBJDIR)/data.o \
       $(OBJDIR)/sqrt_ispc.o \
       $(PPM_OBJ) \
       $(TASKSYS_OBJ)

# 默认目标
default: $(APP_NAME)

# 打印构建信息
info:
	@echo "操作系统: $(UNAME_S)"
	@echo "架构: $(UNAME_M)"
	@echo "C++编译器: $(CXX)"
	@echo "C++标志: $(CXXFLAGS)"
	@echo "ISPC编译器: $(ISPC)"
	@echo "ISPC标志: $(ISPCFLAGS)"

# 伪目标
.PHONY: dirs clean info

# 创建目录
dirs:
	@/bin/mkdir -p $(OBJDIR)/

# 清理
clean:
	@/bin/rm -rf $(OBJDIR) *.ppm *~ $(APP_NAME) 2>/dev/null || true

# 链接应用程序
$(APP_NAME): dirs $(OBJS)
	$(CXX) $(CXXFLAGS) -o $@ $(OBJS) -lm $(TASKSYS_LIB)

# 编译普通.cpp文件
$(OBJDIR)/%.o: %.cpp
	$(CXX) $< $(CXXFLAGS) -c -o $@

# 编译common目录中的.cpp文件
$(OBJDIR)/%.o: $(COMMONDIR)/%.cpp
	$(CXX) $< $(CXXFLAGS) -c -o $@

# main.o的依赖
$(OBJDIR)/main.o: $(OBJDIR)/$(APP_NAME)_ispc.h $(COMMONDIR)/CycleTimer.h

# ISPC编译规则（生成.o和.h文件）
$(OBJDIR)/%_ispc.h $(OBJDIR)/%_ispc.o: %.ispc
	$(ISPC) $(ISPCFLAGS) $< -o $(OBJDIR)/$*_ispc.o -h $(OBJDIR)/$*_ispc.h
