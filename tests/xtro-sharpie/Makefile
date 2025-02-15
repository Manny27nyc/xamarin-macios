TOP=../..

include $(TOP)/Make.config

# a 64bits mono is required because of the clang requirement
MONO ?= mono64 --debug
SHARPIE ?= sharpie

XCODE=$(abspath $(XCODE_DEVELOPER_ROOT)/../..)

ANNOTATIONS_DIR=.
DOTNET_ANNOTATIONS_DIR=api-annotations-dotnet

all-local::

clean-local::
	rm -f xtro-report/bin/Debug/xtro-report.exe bin/Debug/xtro-sharpie.exe
	rm -f *.tmp
	rm -rf *os*.pch*

bin/Debug/xtro-sharpie.exe xtro-report/bin/Debug/xtro-report.exe xtro-sanity/bin/Debug/xtro-sanity.exe build: .stamp-build

.stamp-build: $(wildcard *.cs)
	$(Q_BUILD) $(SYSTEM_MSBUILD) $(MSBUILD_VERBOSITY) xtro-sharpie.sln /r /bl:$@.binlog
	$(Q) touch $@

XIOS ?= $(TOP)/_ios-build/Library/Frameworks/Xamarin.iOS.framework/Versions/git/lib/64bits/iOS/Xamarin.iOS.dll
XIOS_GL ?= $(TOP)/_ios-build/Library/Frameworks/Xamarin.iOS.framework/Versions/Current/lib/mono/Xamarin.iOS/OpenTK-1.0.dll
XIOS_ARCH = arm64
XIOS_PCH = iphoneos$(IOS_SDK_VERSION)-$(XIOS_ARCH).pch
XIOS_RID = ios-arm64

$(XIOS_PCH): .stamp-check-sharpie
	$(SHARPIE) sdk-db --xcode $(XCODE) -s iphoneos$(IOS_SDK_VERSION) -a $(XIOS_ARCH) \
	-exclude RealityKit \
	-i ThreadNetwork/THClient.h \

XWATCHOS ?= $(TOP)/_ios-build/Library/Frameworks/Xamarin.iOS.framework/Versions/git/lib/32bits/watchOS/Xamarin.WatchOS.dll
XWATCHOS_ARCH = armv7
XWATCHOS_PCH = watchos$(WATCH_SDK_VERSION)-$(XWATCHOS_ARCH).pch

$(XWATCHOS_PCH): .stamp-check-sharpie
	$(SHARPIE) sdk-db --xcode $(XCODE) -s watchos$(WATCH_SDK_VERSION) -a $(XWATCHOS_ARCH) -exclude RealityKit

XTVOS ?= $(TOP)/_ios-build/Library/Frameworks/Xamarin.iOS.framework/Versions/git/lib/64bits/tvOS/Xamarin.TVOS.dll
XTVOS_GL ?= $(TOP)/_ios-build/Library/Frameworks/Xamarin.iOS.framework/Versions/git/lib/mono/Xamarin.TVOS/OpenTK-1.0.dll
XTVOS_ARCH = arm64
XTVOS_PCH = appletvos$(TVOS_SDK_VERSION)-$(XTVOS_ARCH).pch
XTVOS_RID = tvos-arm64

$(XTVOS_PCH): .stamp-check-sharpie
	$(SHARPIE) sdk-db --xcode $(XCODE) -s appletvos$(TVOS_SDK_VERSION) -a $(XTVOS_ARCH) -exclude RealityKit

XMACOS ?= $(TOP)/_mac-build/Library/Frameworks/Xamarin.Mac.framework/Versions/git/lib/64bits/mobile/Xamarin.Mac.dll
XMACOS_ARCH = x86_64
XMACOS_PCH = macosx$(MACOS_SDK_VERSION)-$(XMACOS_ARCH).pch
XMACOS_RID = osx-x64

$(XMACOS_PCH): .stamp-check-sharpie
	$(SHARPIE) sdk-db --xcode $(XCODE) -s macosx$(MACOS_SDK_VERSION) -a $(XMACOS_ARCH) -exclude RealityKit -exclude JavaNativeFoundation

XMACCATALYST_ARCH = x86_64
XMACCATALYST_PCH = ios$(MACCATALYST_SDK_VERSION)-macabi-$(XMACCATALYST_ARCH).pch
XMACCATALYST_RID = maccatalyst-x64

$(XMACCATALYST_PCH): .stamp-check-sharpie
	$(SHARPIE) sdk-db --xcode $(XCODE) -s ios$(MACCATALYST_SDK_VERSION)-macabi -a $(XMACCATALYST_ARCH) \
	-exclude AGL \
	-exclude CalendarStore \
	-exclude Carbon \
	-exclude ClockKit \
	-exclude DiscRecordingUI \
	-exclude GLKit \
	-exclude ICADevices \
	-exclude InputMethodKit \
	-exclude InstallerPlugins \
	-exclude IOBluetooth \
	-exclude IOBluetoothUI \
	-exclude JavaNativeFoundation \
	-exclude LDAP \
	-exclude Python \
	-exclude Quartz \
	-exclude QuickLookUI \
	-exclude RealityKit \
	-exclude SecurityInterface \
	-exclude Virtualization \
	-i HomeKit/HomeKit.h \

define DotNetAssembly
X$(2)_DOTNET ?= $(DOTNET_DESTDIR)/Microsoft.$(1).Runtime.$(X$(2)_RID)/runtimes/$(X$(2)_RID)/lib/$(DOTNET_TFM)/$(DOTNET_$(2)_ASSEMBLY_NAME).dll

dotnet-$(1)-$($(2)_SDK_VERSION).g.cs: .stamp-check-sharpie
	$$(SHARPIE) query -bind $$(X$(2)_PCH) > $$@
endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call DotNetAssembly,$(platform),$(shell echo $(platform) | tr a-z A-Z))))

ios-$(IOS_SDK_VERSION).g.cs: $(XIOS_PCH)

ifdef INCLUDE_IOS
gen-ios: ios-$(IOS_SDK_VERSION).g.cs .stamp-check-sharpie
	$(SHARPIE) query -bind $(XIOS_PCH) > ios-$(IOS_SDK_VERSION).g.cs
else
gen-ios: ; @true
endif

tvos-$(TVOS_SDK_VERSION).g.cs: $(XTVOS_PCH)

ifdef INCLUDE_TVOS
gen-tvos: tvos-$(TVOS_SDK_VERSION).g.cs .stamp-check-sharpie
	$(SHARPIE) query -bind $(XTVOS_PCH) > tvos-$(TVOS_SDK_VERSION).g.cs
else
gen-tvos: ; @true
endif

watchos-$(WATCH_SDK_VERSION).g.cs: $(XWATCHOS_PCH)

ifdef INCLUDE_WATCH
gen-watchos: watchos-$(WATCH_SDK_VERSION).g.cs .stamp-check-sharpie
	$(SHARPIE) query -bind $(XWATCHOS_PCH) > watchos-$(WATCH_SDK_VERSION).g.cs
else
gen-watchos: ; @true
endif

macos-$(MACOS_SDK_VERSION).g.cs: $(XMACOS_PCH)

ifdef INCLUDE_MAC
gen-macos: macos-$(MACOS_SDK_VERSION).g.cs .stamp-check-sharpie
	$(SHARPIE) query -bind $(XMACOS_PCH) > macos-$(MACOS_SDK_VERSION).g.cs
else
gen-macos: ; @true
endif

maccatalyst-$(MACCATALYST_SDK_VERSION).g.cs: $(XMACCATALYST_PCH)

gen-all: gen-ios gen-tvos gen-watchos gen-macos

wrench:
	$(MAKE) -j8 classify

dotnet-wrench:
	$(MAKE) -j8 dotnet-classify

report/index.html: xtro-report/bin/Debug/xtro-report.exe .stamp-classify
	rm -rf report/
	$(MONO) xtro-report/bin/Debug/xtro-report.exe $(ANNOTATIONS_DIR) report

report: report/index.html

dotnet-report/index.html: xtro-report/bin/Debug/xtro-report.exe .stamp-dotnet-classify
	rm -rf report-dotnet
	$(MONO) xtro-report/bin/Debug/xtro-report.exe $(DOTNET_ANNOTATIONS_DIR) report-dotnet

dotnet-report: dotnet-report/index.html

report-short:
	JENKINS_SERVER_COOKIE=1 make report

define DotNetClassify
.stamp-dotnet-classify-$(1): bin/Debug/xtro-sharpie.exe $$(X$(2)_PCH) $$(X$(2)_DOTNET)
	rm -f $$(DOTNET_ANNOTATIONS_DIR)/$(1)-*.raw
	$(MONO) bin/Debug/xtro-sharpie.exe --output-directory $$(DOTNET_ANNOTATIONS_DIR) $$(X$(2)_PCH) $$(X$(2)_DOTNET)
	touch $$@

dotnet-classify-$(1): .stamp-dotnet-classify-$(1)
DOTNET_CLASSIFY += .stamp-dotnet-classify-$(1)
endef
$(foreach platform,$(DOTNET_PLATFORMS),$(eval $(call DotNetClassify,$(platform),$(shell echo $(platform) | tr a-z A-Z))))

ifdef INCLUDE_IOS
INCLUDED_PLATFORMS+=iOS
CLASSIFY+=.stamp-classify-ios
.stamp-classify-ios: bin/Debug/xtro-sharpie.exe $(XIOS_PCH) $(XIOS) $(XIOS_GL)
	rm -f $(ANNOTATIONS_DIR)/iOS-*.raw
	$(MONO) bin/Debug/xtro-sharpie.exe --output-directory $(ANNOTATIONS_DIR) $(XIOS_PCH) $(XIOS) $(XIOS_GL)
	$(Q) touch $@
endif

ifdef INCLUDE_TVOS
INCLUDED_PLATFORMS+=tvOS
CLASSIFY+=.stamp-classify-tvos
.stamp-classify-tvos: bin/Debug/xtro-sharpie.exe $(XTVOS_PCH) $(XTVOS) $(XTVOS_GL)
	rm -f $(ANNOTATIONS_DIR)/tvOS-*.raw
	$(MONO) bin/Debug/xtro-sharpie.exe --output-directory $(ANNOTATIONS_DIR) $(XTVOS_PCH) $(XTVOS) $(XTVOS_GL)
	$(Q) touch $@
endif

ifdef INCLUDE_WATCH
INCLUDED_PLATFORMS+=watchOS
CLASSIFY+=.stamp-classify-watchos
.stamp-classify-watchos: bin/Debug/xtro-sharpie.exe $(XWATCHOS_PCH) $(XWATCHOS)
	rm -f $(ANNOTATIONS_DIR)/watchOS-*.raw
	$(MONO) bin/Debug/xtro-sharpie.exe --output-directory $(ANNOTATIONS_DIR) $(XWATCHOS_PCH) $(XWATCHOS)
	$(Q) touch $@
endif

ifdef INCLUDE_MAC
INCLUDED_PLATFORMS+=macOS
CLASSIFY+=.stamp-classify-macos
.stamp-classify-macos: bin/Debug/xtro-sharpie.exe $(XMACOS_PCH) $(XMACOS)
	rm -f $(ANNOTATIONS_DIR)/macOS-*.raw
	$(MONO) bin/Debug/xtro-sharpie.exe --output-directory $(ANNOTATIONS_DIR) $(XMACOS_PCH) $(XMACOS)
	$(Q) touch $@
endif

.stamp-classify: xtro-sanity/bin/Debug/xtro-sanity.exe $(CLASSIFY)
	$(MONO) xtro-sanity/bin/Debug/xtro-sanity.exe $(abspath $(ANNOTATIONS_DIR)) $(INCLUDED_PLATFORMS)
	$(Q) touch $@

classify: .stamp-classify

.stamp-dotnet-classify: xtro-sanity/bin/Debug/xtro-sanity.exe $(DOTNET_CLASSIFY)
	$(MONO) xtro-sanity/bin/Debug/xtro-sanity.exe $(abspath $(DOTNET_ANNOTATIONS_DIR)) $(DOTNET_PLATFORMS)
	$(Q) touch $@

dotnet-classify: .stamp-dotnet-classify

insane:
	XTRO_SANITY_SKIP=1 make all

remove-empty:
	find . -size 0 | xargs git rm

all: report dotnet-report

.stamp-check-sharpie:
	@$(TOP)/system-dependencies.sh --ignore-all --enforce-sharpie
	@touch $@
	
remove-empty-files:
	find . -empty -exec git rm -f {} \;

U2TODO = u2todo/bin/Debug/u2todo.exe
$(U2TODO): $(wildcard u2todo/*.cs u2todo/*.csproj Filter.cs)
	$(Q) $(SYSTEM_MSBUILD) $(MSBUILD_VERBOSITY) /bl:$@.binlog /r
	$(Q) touch $@

unclassified2todo: $(U2TODO)
	cd $(ANNOTATIONS_DIR) && $(MONO) $(abspath $(U2TODO))
	@for filename in $(git status -s | cut -c4- | grep .todo); do \
		sort -o "$filename" "$filename"; \
	done

unclassified2todo-dotnet: $(U2TODO)
	$(SYSTEM_CSC) -features:strict Filter.cs u2todo/u2todo.cs
	cd $(DOTNET_ANNOTATIONS_DIR) && $(MONO) $(abspath $(U2TODO))
