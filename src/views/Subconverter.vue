<template>
  <div>
    <el-row style="margin-top: 10px;">
      <el-col>
        <el-card style="margin-top:20px;max-width:800px;margin:auto;opacity:0.8;blackground-color:#0F4677;border-radius: 20px;">
          <div slot="header" style="blackground-color:#0F4677;text-align:center;font-size :25px !important;font-weight: bold !important;">
            <svg-icon icon-class="lock" style="margin-left: 20px" title="完整魔改版:v1.2"/>
            つつの订阅转换
            <svg-icon icon-class="telegram" style="margin-left: 10px" title="加入Telegram吹水群" @click="gotoTgChannel" />
          </div>
          <el-container>
            <el-form :model="form" label-width="80px" label-position="left" style="width: 100%;">
              <el-form-item label="进阶选项:">
                
                  <div class="switch">
                    <input id="cmn-toggle-1" class="cmn-toggle cmn-toggle-round" type="checkbox">
                    <label for="cmn-toggle-1"></label>
                  </div>
                  <div class="switch">
                    <input id="cmn-toggle-4" click = "selectMode()" class="cmn-toggle cmn-toggle-round-flat" type="checkbox">
                    <label for="cmn-toggle-4" ></label>
                  </div>
                  <div class="switch">
                    <input id="cmn-toggle-7" class="cmn-toggle cmn-toggle-yes-no" type="checkbox">
                    <label for="cmn-toggle-7" class="cmn-toggle-label" data-on="1" data-off="2"></label>
                  </div>

                <el-radio v-model="advanced" label="1">基础模式</el-radio>
                <el-radio v-model="advanced" label="2">进阶模式</el-radio>
              </el-form-item>
              <el-form-item label="订阅链接:">
                <el-input
                  v-model="form.sourceSubUrl"
                  type="textarea"
                  rows="3"
                  placeholder="支持订阅或ss/ssr/vmess/trojan链接，多个链接每行一个或用 | 分隔"
                  @blur="saveSubUrl"
                />
              </el-form-item>
              <el-form-item label="生成类型:">
                <el-select v-model="form.clientType" style="width: 100%">
                  <el-option v-for="(v, k) in options.clientTypes" :key="k" :label="k" :value="v"></el-option>
                </el-select>
              </el-form-item>


                <el-form-item label="远程配置:">
                  <el-select
                    v-model="form.remoteConfig"
                    allow-create
                    filterable
                    placeholder="请选择"
                    style="width: 100%"
                  >
                    <el-option-group
                      v-for="group in options.remoteConfig"
                      :key="group.label"
                      :label="group.label"
                    >
                      <el-option
                        v-for="item in group.options"
                        :key="item.value"
                        :label="item.label"
                        :value="item.value"
                      ></el-option>
                    </el-option-group>
                </el-select>
              </el-form-item>

              <el-form-item label="后端地址:">

              <el-select
                  v-model="form.customBackend"
                  allow-create
                  filterable
                  placeholder="请选择"
                  style="width: 100%"
                >
                  <el-option v-for="(v, k) in options.customBackend" :key="k" :label="k" :value="v"></el-option>

                </el-select>

              </el-form-item>

              <div v-if="advanced === '2'">


                <el-form-item label="包含节点:">
                  <el-input v-model="form.includeRemarks" placeholder="节点名包含的关键字，支持正则" />
                </el-form-item>
                <el-form-item label="排除节点:">
                  <el-input v-model="form.excludeRemarks" placeholder="节点名不包含的关键字，支持正则" />
                </el-form-item>
                <el-form-item label="输出名称:">
                  <el-input v-model="form.filename" placeholder="返回的订阅文件名" />
                </el-form-item>
                <el-form-item label-width="0px">
                  <el-row type="flex">
                    <el-col>
                      <el-checkbox v-model="form.nodeList" label="输出为 Node List" border></el-checkbox>
                      <el-checkbox v-model="form.emoji" label="Emoji" border></el-checkbox>
                      <el-checkbox v-model="form.sort" label="排序节点" border></el-checkbox>
                      <el-checkbox v-model="form.udp" @change="needUdp = true" label="启用 UDP" border></el-checkbox>
                      <el-checkbox v-model="form.tpl.surge.doh" label="Surge.DoH" border></el-checkbox>
                      <el-checkbox v-model="form.tpl.clash.doh" label="Clash.DoH" border></el-checkbox>
                    </el-col>
                  </el-row>
                </el-form-item>
              </div>

              <div style="margin-top: 50px"></div>

              <el-divider content-position="center">
                <i class="el-icon-magic-stick"></i>
              </el-divider>

              <el-form-item label="定制订阅:">
                <el-input class="copy-content" disabled v-model="customSubUrl">
                  <el-button
                    slot="append"
                    v-clipboard:copy="customSubUrl"
                    v-clipboard:success="onCopy"
                    ref="copy-btn"
                    icon="el-icon-document-copy"
                  >复制</el-button>
                </el-input>
              </el-form-item>
              <el-form-item label="订阅短链:">
                <el-input class="copy-content" disabled v-model="curtomShortSubUrl">
                  <el-button
                    slot="append"
                    v-clipboard:copy="curtomShortSubUrl"
                    v-clipboard:success="onCopy"
                    ref="copy-btn"
                    icon="el-icon-document-copy"
                  >复制</el-button>
                </el-input>
              </el-form-item>

              
              <el-form-item label-width="0px" style="margin-top: 40px; text-align: center">
                <el-button
                  style="width: 120px"
                  type="primary"
                  @click="makeUrl"
                  :disabled="form.sourceSubUrl.length === 0"
                >生成订阅链接</el-button>
                <el-button
                  style="width: 120px"
                  type="primary"
                  @click="makeShortUrl"
                  :loading="loading"
                  :disabled="customSubUrl.length === 0"
                >生成短链接</el-button>
                <!-- <el-button style="width: 120px" type="primary" @click="surgeInstall" icon="el-icon-connection">一键导入Surge</el-button> -->
              </el-form-item>

              <el-form-item label-width="0px" style="text-align: center">
                <el-button
                  style="width: 120px"
                  type="primary"
                  @click="dialogUploadConfigVisible = true"
                  icon="el-icon-upload"
                  :loading="loading"
                >上传配置</el-button>
                <el-button
                  style="width: 120px"
                  type="primary"
                  @click="clashInstall"
                  icon="el-icon-connection"
                  :disabled="customSubUrl.length === 0"
                >一键导入Clash</el-button>
              </el-form-item>
            </el-form>
          </el-container>
        </el-card>
      </el-col>
    </el-row>

    <el-dialog
      :visible.sync="dialogUploadConfigVisible"
      :show-close="false"
      :close-on-click-modal="false"
      :close-on-press-escape="false"
      width="700px"
    >
      <div slot="title">
        Remote config upload
        <el-popover trigger="hover" placement="right" style="margin-left: 10px">
          <el-link type="primary" :href="sampleConfig" target="_blank" icon="el-icon-info">参考配置</el-link>
          <i class="el-icon-question" slot="reference"></i>
        </el-popover>
      </div>
      <el-form label-position="left">
        <el-form-item prop="uploadConfig">
          <el-input
            v-model="uploadConfig"
            type="textarea"
            :autosize="{ minRows: 15, maxRows: 15}"
            maxlength="10000"
            show-word-limit
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="uploadConfig = ''; dialogUploadConfigVisible = false">取 消</el-button>
        <el-button
          type="primary"
          @click="confirmUploadConfig"
          :disabled="uploadConfig.length === 0"
        >确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
const project = process.env.VUE_APP_PROJECT
const remoteConfigSample = process.env.VUE_APP_SUBCONVERTER_REMOTE_CONFIG
const gayhubRelease = process.env.VUE_APP_BACKEND_RELEASE
const defaultBackend = process.env.VUE_APP_SUBCONVERTER_DEFAULT_BACKEND + '/sub?'
const shortUrlBackend = process.env.VUE_APP_MYURLS_DEFAULT_BACKEND + '/short'
const configUploadBackend = process.env.VUE_APP_CONFIG_UPLOAD_BACKEND + '/config/upload'
const tgBotLink = process.env.VUE_APP_BOT_LINK

export default {
  data() {
    var data = {
      backendVersion: '0.6.4',
      advanced: "1",

      // 是否为 PC 端
      isPC: true,

      options: {
        clientTypes: {
          Clash: "clash",
          ClashR: "clashr",
          Surge2: "surge&ver=2",
          Surge3: "surge&ver=3",
          Surge4: "surge&ver=4",
          Quantumult: "quan",
          "Quantumult X": "quanx",
          Loon: "loon",
          Mellow: "mellow",
          Surfboard: "surfboard",
          "Shadowsocks(SIP002)": "ss",
          "Shadowsocks Android(SIP008)": "sssub",
          ShadowsocksR: "ssr",
          ShadowsocksD: "ssd",          
          V2Ray: "v2ray",
          Trojan: "trojan",
          "混合订阅（mixed）": "mixed",
          "自动判断客户端": "auto",
        },
        customBackend: {
          "api.tsutsu.cc (つつ提供-香港CN2稳定)": "https://api.tsutsu.cc/sub?",
          "api2.tsutsu.cc (つつ提供-香港CN2备用)": "https://api2.tsutsu.cc/sub?",
          "api.v1.mk（肥羊提供-四端八核负载)": "https://api.v1.mk/sub?",
          "subcon.dlj.tf (subconverter作者提供) ": "https://subcon.dlj.tf/sub?",
          "api.dler.io (sub作者&lhie1提供)": "https://api.dler.io/sub?",
          "api.wcc.best (sub-web作者提供)": "https://api.wcc.best/sub?",
          "api.hope140.live (hope提供-vercel)": "https://api.hope140.live/sub?",
          "sub.id9.cc (品云提供)": "https://sub.id9.cc/sub?",
        },
        backendOptions: [
          { value: "https://api.tsutsu.cc/sub?" },
          { value: "https://api2.tsutsu.cc/sub?" },
          { value: "https://api.v1.mk/sub?" },
          { value: "https://subcon.dlj.tf/sub?" },
          { value: "https://api.dler.io/sub?" },
          { value: "https://api.wcc.best/sub?" },
          { value: "https://api.hope140.live/sub?" },
          { value: "https://sub.id9.cc/sub?" },
        ],
        remoteConfig: [
          {
            label: "つつ自用,投稿请tg找 @Ox208",
            options: [
              {
                label: "つつ自用-完整分组",
                value:
                  "https://cdn.staticaly.com/gh/lhl77/sub-ini/main/tsutsu-full.ini"
              },
              {
                label: "つつ自用-完整分组(地区自动选择)",
                value:
                  "https://cdn.staticaly.com/gh/lhl77/sub-ini/main/tsutsu-full-urltest.ini"
              },
              {
                label: "つつ自用-Immtel专用(地区自动选择)",
                value:
                  "https://cdn.staticaly.com/gh/lhl77/sub-ini/main/tsutsu-full-urltest-imm.ini"
              },
              {
                label: "つつ自用-超jb精简分组(含国内分流)",
                value:
                  "https://cdn.staticaly.com/gh/lhl77/sub-ini/main/tsutsu-mini-gfw.ini"
              },
            ]
          },
          {
            label: "用户投稿",
            options: [
              {
                label: "hope140自用配置 (与Github同步)",
                value:
                  "https://cdn.staticaly.com/gh/hope140/Clash/beta/hope140.yaml"
              },
              {
                label: "hope140去广告配置",
                value:
                  "https://cdn.staticaly.com/gh/hope140/Clash/beta/Adblock.yaml"
              },
              {
                label: "hope140全分组",
                value:
                  "https://cdn.staticaly.com/gh/hope140/Clash/beta/All.yaml"
              },
              {
                label: "Nine499自用规则",
                value:
                  "https://cdn.staticaly.com/gh/Nine499/Clash-Rule/master/Rule"
              },
              {
                label: "AllenXu精简版多国家",
                value:
                  "https://raw.githubusercontent.com/hyt-allen-xu/webcdn/master/cdn_multicountry.ini"
              },
              {
                label: "AllenXu小机场专用",
                value:
                  "https://raw.githubusercontent.com/hyt-allen-xu/webcdn/master/smallairport.ini"
              },
            ]
          },
          {
            label: "ACL4SSR",
            options: [
              {
                label: "ACL4SSR默认",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online.ini"
              },
              {
                label: "ACL4SSR去广告",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_AdblockPlus.ini"
              },
              {
                label: "ACL4SSR无自动测速",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoAuto.ini"
              },
              {
                label: "ACL4SSR无广告拦截",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_NoReject.ini"
              },
              {
                label: "ACL4SSR精简版",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini.ini"
              },
              {
                label: "ACL4SSR精简去广告",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_AdblockPlus.ini"
              },
              {
                label: "ACL4SSR精简多重模式",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Mini_MultiMode.ini"
              },
              {
                label: "ACL4SSR精简版带港美日国家",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Mini_MultiCountry.ini"
              },
              {
                label: "ACL4SSR全分组",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full.ini"
              },
              {
                label: "ACL4SSR全分组多模式",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_MultiMode.ini"
              },
              {
                label: "ACL4SSR全分组重度用户",
                value:
                  "https://cdn.staticaly.com/gh/ACL4SSR/ACL4SSR/master/Clash/config/ACL4SSR_Online_Full_Netflix.ini"
              }
            ]
          },
          {
            label: "特殊",
            options: [
              {
                label: "基础无规则",
                value:
                  "https://subconverter.oss-ap-southeast-1.aliyuncs.com/Rules/RemoteConfig/special/basic.ini"
              }
            ]
          }
        ]
      },
      form: {
        sourceSubUrl: "",
        clientType: "",
        customBackend: "",
        remoteConfig: "",
        excludeRemarks: "",
        includeRemarks: "",
        filename: "",
        emoji: true,
        nodeList: false,
        extraset: false,
        sort: false,
        udp: false,
        tfo: false,
        scv: false,
        expand: true, // 是否将规则全文写进配置文件
        fdn: false,
        appendType: false,
        insert: false, // 是否插入默认订阅的节点，对应配置项 insert_url
        new_name: true, // 是否使用 Clash 新字段
        // tpl 定制功能
        tpl: {
          surge: {
            doh: false // dns 查询是否使用 DoH
          },
          clash: {
            doh: false
          }
        }
      },

      loading: false,
      customSubUrl: "",
      curtomShortSubUrl: "",

      dialogUploadConfigVisible: false,
      uploadConfig: "",
      uploadPassword: "",
      myBot: tgBotLink,
      sampleConfig: remoteConfigSample,

      needUdp: false, // 是否需要添加 udp 参数
    };

    // window.console.log(data.options.remoteConfig);
    // window.console.log(data.options.customBackend);
    let phoneUserAgent = /Android|webOS|iPhone|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
      navigator.userAgent
    );

    if (phoneUserAgent) {
      let acl4ssrConfig = data.options.remoteConfig[1].options;
      for (let i = 0; i < acl4ssrConfig.length; i++) {
        if (acl4ssrConfig[i].label.length > 10) {
          acl4ssrConfig[i].label = acl4ssrConfig[i].label.replace(/\s.*/, "");
        }
      }
      var serverList = {};
      let serverKeys = Object.keys(data.options.customBackend);
      for (let i = 0; i < serverKeys.length; i++) {
        let key = serverKeys[i].replace(/\(.*/, "");
        serverList[key] = data.options.customBackend[serverKeys[i]];
      }
      data.options.customBackend = serverList;
    }
    return data;
  },
  created() {
    // document.title = "Subscription Converter";
    document.title = "つつの订阅转换 ";
     this.isPC = this.$getOS().isPc;

    // 获取 url cache
    if (process.env.VUE_APP_USE_STORAGE === 'true') {
      this.form.sourceSubUrl = this.getLocalStorageItem('sourceSubUrl')
    }
  },
  mounted() {
    this.form.clientType = "clash";
    this.form.customBackend = "https://api.tsutsu.cc/sub?";
    this.form.remoteConfig = "https://cdn.staticaly.com/gh/lhl77/sub-ini/main/tsutsu-full.ini";
    //this.getBackendVersion();
  },
  methods: {
    onCopy() {
      this.$message.success("Copied!");
    },
    goToProject() {
      window.open(project);
    },
	gotoTgChannel() {
      window.open(tgBotLink);
    },
    gotoGayhub() {
      window.open(gayhubRelease);
    },
    gotoRemoteConfig() {
      window.open(remoteConfigSample);
    },
    clashInstall() {
      if (this.customSubUrl === "") {
        this.$message.error("请先填写必填项，生成订阅链接");
        return false;
      }

      const url = "clash://install-config?url=";
      window.open(
        url +
          encodeURIComponent(
            this.curtomShortSubUrl !== ""
              ? this.curtomShortSubUrl
              : this.customSubUrl
          )
      );
    },
    surgeInstall() {
      if (this.customSubUrl === "") {
        this.$message.error("请先填写必填项，生成订阅链接");
        return false;
      }

      const url = "surge://install-config?url=";
      window.open(url + this.customSubUrl);
    },
    makeUrl() {
      if (this.form.sourceSubUrl === "" || this.form.clientType === "") {
        this.$message.error("订阅链接与客户端为必填项");
        return false;
      }
      // 远程接口
      let backend =
        this.form.customBackend === ""
          ? defaultBackend
          : this.form.customBackend;

      // 远程配置
      let config = this.form.remoteConfig === "" ? "" : this.form.remoteConfig;

      let sourceSub = this.form.sourceSubUrl;
      sourceSub = sourceSub.replace(/(\n|\r|\n\r)/g, "|");

      // 薯条屏蔽
      if (sourceSub.indexOf("losadhwse") !== -1 && (backend.indexOf("py6.pw") !== -1 || backend.indexOf("subconverter-web.now.sh") !== -1 || backend.indexOf("subconverter.herokuapp.com") !== -1 || backend.indexOf("api.wcc.best") !== -1)) {
        this.$alert('此机场订阅已将该后端屏蔽，请自建后端转换。', '转换错误提示', {
          confirmButtonText: '确定',
          callback: action => {
            this.$message({
              type: 'error',
              message: `action: ${ action }`
            });
          }
        });
        return false;
      }

      this.customSubUrl =
        backend +
        "target=" +
        this.form.clientType +
        "&url=" +
        encodeURIComponent(sourceSub) +
        "&insert=" +
        this.form.insert;

      if (config !== "") {
        this.customSubUrl += "&config=" + encodeURIComponent(config);
      }

      if (this.advanced === "2") {
        if (this.form.excludeRemarks !== "") {
          this.customSubUrl +=
            "&exclude=" + encodeURIComponent(this.form.excludeRemarks);
        }
        if (this.form.includeRemarks !== "") {
          this.customSubUrl +=
            "&include=" + encodeURIComponent(this.form.includeRemarks);
        }
        if (this.form.filename !== "") {
          this.customSubUrl +=
            "&filename=" + encodeURIComponent(this.form.filename);
        }
        if (this.form.appendType) {
          this.customSubUrl +=
            "&append_type=" + this.form.appendType.toString();
        }

        this.customSubUrl +=
          "&emoji=" +
          this.form.emoji.toString() +
          "&list=" +
          this.form.nodeList.toString() +
          "&tfo=" +
          this.form.tfo.toString() +
          "&scv=" +
          this.form.scv.toString() +
          "&fdn=" +
          this.form.fdn.toString() +
          "&sort=" +
          this.form.sort.toString() +
          "&expand=" +
          this.form.expand.toString();

        if (this.needUdp) {
          this.customSubUrl += "&udp=" + this.form.udp.toString()
        }
        if (this.form.tpl.surge.doh === true) {
          this.customSubUrl += "&surge.doh=true";
        }
        if (this.form.clientType === "clash") {
          if (this.form.tpl.clash.doh === true) {
            this.customSubUrl += "&clash.doh=true";
          }
          this.customSubUrl += "&new_name=" + this.form.new_name.toString();
        }
      }
      this.$copyText(this.customSubUrl);
      this.$message.success("定制订阅已复制到剪贴板");
    },
    makeShortUrl() {
      if (this.customSubUrl === "") {
        this.$message.warning("请先生成订阅链接，再获取对应短链接");
        return false;
      }

      this.loading = true;

      let data = new FormData();
      data.append("longUrl", btoa(this.customSubUrl));

      this.$axios
        .post(shortUrlBackend, data, {
          header: {
            "Content-Type": "application/form-data; charset=utf-8"
          }
        })
        .then(res => {
          if (res.data.Code === 1 && res.data.ShortUrl !== "") {
            this.curtomShortSubUrl = res.data.ShortUrl;
            this.$copyText(res.data.ShortUrl);
            this.$message.success("短链接已复制到剪贴板");
          } else {
            this.$message.error("短链接获取失败：" + res.data.Message);
          }
        })
        .catch(() => {
          this.$message.error("短链接获取失败");
        })
        .finally(() => {
          this.loading = false;
        });
    },
    confirmUploadConfig() {
      if (this.uploadConfig === "") {
        this.$message.warning("远程配置不能为空");
        return false;
      }

      this.loading = true;

      let data = new FormData();
      data.append("password", this.uploadPassword);
      data.append("config", this.uploadConfig);

      this.$axios
        .post(configUploadBackend, data, {
          header: {
            "Content-Type": "application/form-data; charset=utf-8"
          }
        })
        .then(res => {
          if (res.data.Code === 1 && res.data.url !== "") {
            this.$message.success(
              "远程配置上传成功，配置链接已复制到剪贴板，有效期三个月望知悉"
            );

            // 自动填充至『表单-远程配置』
            this.form.remoteConfig = res.data.Url;
            this.$copyText(this.form.remoteConfig);

            this.dialogUploadConfigVisible = false;
          } else {
            this.$message.error("远程配置上传失败：" + res.data.Message);
          }
        })
        .catch(() => {
          this.$message.error("远程配置上传失败");
        })
        .finally(() => {
          this.loading = false;
        });
    },
    backendSearch(queryString, cb) {
      let backends = this.options.backendOptions;

      let results = queryString
        ? backends.filter(this.createFilter(queryString))
        : backends;

      // 调用 callback 返回建议列表的数据
      cb(results);
    },
    createFilter(queryString) {
      return candidate => {
        return (
          candidate.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0
        );
      };
    },
    getBackendVersion() {
      this.$axios
        .get(
          defaultBackend.substring(0, defaultBackend.length - 5) + "/version"
        )
        .then(res => {
          this.backendVersion = res.data.replace(/backend\n$/gm, "");
          this.backendVersion = this.backendVersion.replace("subconverter", "");
        });
    },
    saveSubUrl() {
      if (this.form.sourceSubUrl !== '') {
        this.setLocalStorageItem('sourceSubUrl', this.form.sourceSubUrl)
      }
    },
    getLocalStorageItem(itemKey) {
      const now = +new Date()
      let ls = localStorage.getItem(itemKey)

      let itemValue = ''
      if (ls !== null) {
        let data = JSON.parse(ls)
        if (data.expire > now) {
          itemValue = data.value
        } else {
          localStorage.removeItem(itemKey)
        }
      }

      return itemValue
    },
    setLocalStorageItem(itemKey, itemValue) {
      const ttl = process.env.VUE_APP_CACHE_TTL
      const now = +new Date()

      let data = {
        setTime: now,
        ttl: parseInt(ttl),
        expire: now + ttl * 1000,
        value: itemValue
      }
      localStorage.setItem(itemKey, JSON.stringify(data))
    }
  },

};

</script>
