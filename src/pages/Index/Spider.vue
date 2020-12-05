<template>
    <div class="spiderBox fixScrollY">
        <el-form :model="form" size="mini" label-position="right" label-width="120px">
            <el-form-item label="规则名称：">
                <el-input placeholder="规则名称" type="text" v-model="form.ruleName"></el-input>
            </el-form-item>
            <el-form-item label="">
                <el-button @click="loadRuleFile" type="warning">载入规则</el-button>
                <el-button @click="saveRuleFile" :disabled="!form.ruleName" type="danger">保存规则</el-button>
                <el-button style="float:right;" @click="runRuleFile" :disabled="!form.ruleName" type="primary">执行规则</el-button>
            </el-form-item>
            <el-collapse accordion>
                <el-collapse-item title="第一步：" :disabled="!form.ruleName">
                    <template slot="title">
                        <el-tag type="info" v-if="!form.ruleName" effect="dark"><i class="el-icon-remove"></i> 第一步</el-tag>
                        <el-tag type="warning" v-if="form.ruleName && urlList.length===0" effect="dark"><i class="el-icon-warning"></i> 第一步
                        </el-tag>
                        <el-tag type="success" v-if="form.ruleName && urlList.length>0" effect="dark"><i class="el-icon-success"></i> 第一步
                        </el-tag>
                    </template>
                    <el-form-item label="网址：">
                        <el-input :readonly="loading" placeholder="请输入网址" v-model="form.url" class="input-with-select">
                            <el-select style="width:80px;" v-model="form.method" slot="prepend" placeholder="请选择">
                                <el-option label="GET" value="GET"></el-option>
                                <el-option label="POST" value="POST"></el-option>
                            </el-select>
                        </el-input>
                    </el-form-item>
                    <el-row :gutter="20">
                        <el-col :span="8">
                            <el-form-item label="端口：">
                                <el-input-number
                                        :precision="0"
                                        :min="1"
                                        :step="1"
                                        :step-strictly="true"
                                        :max="65535"
                                        :controls="false"
                                        :readonly="loading"
                                        v-model.number="form.port"
                                        placeholder="80"
                                ></el-input-number>
                            </el-form-item>
                        </el-col>
                        <el-col :span="8">
                            <el-form-item label="超时时间(毫秒)：">
                                <el-input :readonly="loading" v-model.number="form.timeout" placeholder="如果图片下载不完整请将此数值调大"></el-input>
                            </el-form-item>
                        </el-col>
                        <el-col :span="8">
                            <el-form-item label="页面编码：">
                                <el-select :disabled="loading" v-model.number="form.encoding" allow-create filterable placeholder="页面编码">
                                    <el-option label="gb2312" value="gb2312"></el-option>
                                    <el-option label="utf8" value="utf8"></el-option>
                                </el-select>
                            </el-form-item>
                        </el-col>
                    </el-row>
                    <el-form-item label="列表HTML：">
                        <el-input readonly v-model="listResult" type="textarea" placeholder="获取列表HTML"></el-input>
                    </el-form-item>
                    <el-row :gutter="20">
                        <el-col :span="12">
                            <el-form-item label="列表开始代码：">
                                <el-input :readonly="loading" v-model.number="form.listStart" placeholder="列表区域开始代码"></el-input>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="列表结束代码：">
                                <el-input :readonly="loading" v-model.number="form.listEnd" placeholder="列表区域结束代码"></el-input>
                            </el-form-item>
                        </el-col>
                    </el-row>
                    <el-row :gutter="20">
                        <el-col :span="12">
                            <el-form-item label="链接开始代码：">
                                <el-input :readonly="loading" v-model.number="form.urlStart" placeholder="链接开始代码"></el-input>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="链接结束代码：">
                                <el-input :readonly="loading" v-model.number="form.urlEnd" placeholder="链接结束代码"></el-input>
                            </el-form-item>
                        </el-col>
                        <el-row :gutter="20">
                            <el-col :span="12">
                                <el-form-item label="通配符开始：">
                                    <el-input-number
                                            :precision="0"
                                            :min="1"
                                            :step="1"
                                            :step-strictly="true"
                                            :max="65535"
                                            :controls="true"
                                            :readonly="loading"
                                            v-model.number="form.wildcardStart"
                                            placeholder="通配符开始"
                                    ></el-input-number>
                                </el-form-item>
                            </el-col>
                            <el-col :span="12">
                                <el-form-item label="通配符结束：">
                                    <el-input-number
                                            :precision="0"
                                            :min="1"
                                            :step="1"
                                            :step-strictly="true"
                                            :max="65535"
                                            :controls="true"
                                            :readonly="loading"
                                            v-model.number="form.wildcardEnd"
                                            placeholder="通配符结束"
                                    ></el-input-number>
                                </el-form-item>
                            </el-col>
                        </el-row>
                    </el-row>
                    <el-form-item>
                        <el-button :disabled="loading" type="danger" @click="getList">获取列表</el-button>
                        <el-button v-if="loading" type="info" @click="cancel">取消</el-button>
                    </el-form-item>
                    <el-form-item label="列表URL：">
                        <el-table
                                height="250"
                                size="mini"
                                border
                                :data="urlList"
                                v-loading="loading"
                                :element-loading-text="getListLoadingText"
                        >
                            <el-table-column type="index" label="#" align="center"></el-table-column>
                            <el-table-column prop="url" label="列表URL">
                                <template slot-scope="scope">
                                    <el-button type="text" style="padding:0;" @click="openUrl(scope.row.url)">{{scope.row.url}}</el-button>
                                </template>
                            </el-table-column>
                        </el-table>
                    </el-form-item>
                </el-collapse-item>
                <el-collapse-item title="第二步：" :disabled="urlList.length===0">
                    <template slot="title">
                        <el-tag type="info" v-if="urlList.length===0" effect="dark"><i class="el-icon-remove"></i> 第二步</el-tag>
                        <el-tag type="warning" v-if="urlList.length > 0 && tempImgList.length===0" effect="dark">
                            <i class="el-icon-warning"></i> 第二步
                        </el-tag>
                        <el-tag type="success" v-if="tempImgList.length>0" effect="dark"><i class="el-icon-success"></i> 第二步</el-tag>
                    </template>
                    <el-form-item label="图片页HTML：">
                        <el-input readonly v-model="imgSrcResult" type="textarea" placeholder="图片页HTML"></el-input>
                    </el-form-item>
                    <el-form-item label="图片链接前缀：">
                        <el-input :readonly="loading" v-model.number="form.imgServ" placeholder="图片地址前缀"></el-input>
                    </el-form-item>
                    <el-row :gutter="20">
                        <el-col :span="12">
                            <el-form-item label="图片开始代码：">
                                <el-input :readonly="loading" v-model.number="form.imgSrcStart" placeholder="列表开始代码"></el-input>
                            </el-form-item>
                        </el-col>
                        <el-col :span="12">
                            <el-form-item label="图片结束代码：">
                                <el-input :readonly="loading" v-model.number="form.imgSrcEnd" placeholder="列表结束代码"></el-input>
                            </el-form-item>
                        </el-col>
                    </el-row>
                    <el-form-item>
                        <el-button type="danger" :disabled="getImgListLoading" @click="getImgList(urlList)">获取图片地址</el-button>
                        <el-button v-if="getImgListLoading" type="info" @click="cancel">取消</el-button>
                    </el-form-item>
                    <el-form-item label="待下载图片：">
                        <el-table
                                height="250"
                                size="mini"
                                border
                                :data="tempImgList"
                                v-loading="getImgListLoading"
                                :element-loading-text="getImgListLoadingText"
                        >
                            <el-table-column type="index" label="#" align="center"></el-table-column>
                            <el-table-column prop="src" label="图片地址">
                                <template slot-scope="scope">
                                    <el-button type="text" style="padding:0;" @click="openUrl(scope.row.src)">{{scope.row.src}}</el-button>
                                </template>
                            </el-table-column>
                            <el-table-column prop="isDownload" align="center" width="85" label="状态">
                                <template slot-scope="scope">
                                    <el-tag size="mini" effect="dark" v-if="scope.row.isDownload===1" type="info">未下载</el-tag>
                                    <el-tag size="mini" effect="dark" v-if="scope.row.isDownload===2" type="warning">下载中..</el-tag>
                                    <el-tag size="mini" effect="dark" v-if="scope.row.isDownload===3" type="success">下载成功</el-tag>
                                    <el-tag size="mini" effect="dark" v-if="scope.row.isDownload===4" type="danger">下载失败</el-tag>
                                </template>
                            </el-table-column>
                        </el-table>
                    </el-form-item>
                </el-collapse-item>
                <el-collapse-item title="第三步：" :disabled="tempImgList.length===0 || getImgListLoading">
                    <template slot="title">
                        <el-tag type="info" v-if="(tempImgList.length===0 || getImgListLoading) && imgList.length==0" effect="dark">
                            <i class="el-icon-remove"></i> 第三步
                        </el-tag>
                        <el-tag
                                type="warning"
                                v-if="!getImgListLoading && tempImgList.length>0 && imgList.length<tempImgList.length"
                                effect="dark"
                        ><i class="el-icon-warning"></i> 第三步
                        </el-tag>
                        <el-tag type="success" v-if="imgList.length>=tempImgList.length && imgList.length!==0" effect="dark">
                            <i class="el-icon-success"></i> 第三步
                        </el-tag>
                    </template>
                    <el-form-item label="保存目录：">
                        <el-input readonly placeholder="请输入网址" v-model="defaultSaveImgPath" class="input-with-select">
                            <el-button
                                    style="background:#409EFF;color:#fff;border-radius: 0 3px 3px 0;border:1px solid #409EFF;"
                                    @click="choosePath"
                                    slot="append"
                            >选择
                            </el-button>
                        </el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button
                                type="danger"
                                @click="downloadPictureByList(tempImgList)"
                                size="mini"
                                :disabled="tempImgList.length===0 || loading===true"
                        >下载图片
                        </el-button>
                        <el-button v-if="loading===true" type="info" @click="cancel">取消</el-button>
                    </el-form-item>
                    <el-form-item label="已下载图片：">
                        <div style="margin-bottom:20px;">
                        </div>
                        <div v-if="true">
                            <el-image
                                    class="downloadImg"
                                    v-for="item in imgList"
                                    :preview-src-list="imgList"
                                    :src="item"
                                    fit="cover"
                            ></el-image>
                            <div v-if="percentage>0 && tempImgList.length" class="downloadImgPlaceholder">
                                <el-progress
                                        :width="80"
                                        :color="colors"
                                        type="circle"
                                        :show-text="true"
                                        :format="progressText"
                                        :percentage="percentage"
                                ></el-progress>
                            </div>
                            <div v-for="o in tempImgList.length" class="downloadImgPlaceholder" v-show="o>imgList.length+(loading?1:0)">
                                <el-progress
                                        :width="80"
                                        :color="colors"
                                        :format="progressText"
                                        type="circle"
                                        :show-text="true"
                                        :percentage="0"
                                ></el-progress>
                            </div>
                        </div>
                    </el-form-item>
                </el-collapse-item>
            </el-collapse>
        </el-form>
        <el-dialog
            title="执行任务中"
            :visible.sync="dialogVisible"
            width="300px"
            :fullscreen="false"
            :close-on-click-modal="false"
            :close-on-press-escape="false"
            :show-close="false"
        >
            <div>
                {{ dialogText }}
                <el-progress :width="100" :percentage="parseInt(dialogPercentage)" ></el-progress>
            </div>
            <span slot="footer" class="dialog-footer">
                <el-button size="mini" type="primary" @click="cancel">取消任务</el-button>
            </span>
        </el-dialog>
    </div>
</template>
<script>
    export default {
        name: 'Spider',
        data() {
            return {
                loading: false,
                loadingText: 'loading',
                getListLoadingText: '正在获取列表...',
                dialogVisible: false,
                dialogText: 'loading',
                dialogPercentage: 0,
                defaultSaveImgPath: '',
                isCancel: false, // 是否中止任务
                form: {
                    ruleName: '',
                    url: '',
                    method: 'GET',
                    port: 80,
                    wildcardStart: 1,
                    wildcardEnd: 9,
                    encoding: 'utf8',
                    timeout: 1000,
                    path: '',
                    imgServ: '',
                    listStart: '',
                    listEnd: '',
                    urlStart: '',
                    urlEnd: '',
                    imgSrcStart: '',
                    imgSrcEnd: '',
                },
                result: '',
                listResult: '',
                imgSrcResult: '',
                imgList: [],
                tempImgList: [],
                urlList: [],
                timmer: null,
                timmer2: null,
                fullLoading: null,
                percentage: 0,
                getImgListLoading: false,
                getImgListLoadingText: '',
                colors: [
                    {color: '#f56c6c', percentage: 20},
                    {color: '#e6a23c', percentage: 40},
                    {color: '#5cb87a', percentage: 60},
                    {color: '#1989fa', percentage: 80},
                    {color: '#6f7ad3', percentage: 100}
                ],
            }
        },
        created() {
            this.defaultSaveImgPath = localStorage.getItem('defaultSaveImgPath') ? localStorage.getItem('defaultSaveImgPath') : os.homedir() + '\\Downloads';
        },
        methods: {
            openUrl(url) {
                electron.shell.openExternal(url)
            },
            getListByUrl (url, callback) {

                let header = {
                    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
                    "Accept-Encoding": "gzip, deflate, br",
                    "Accept-Language": "zh-CN,zh;q=0.9",
                    "Cache-Control": "no-cache",
                    "Connection": "keep-alive",
                    "referer": url,
                    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.105 Safari/537.36"
                };
                this.loading = true;
                const request = new electron.remote.net.request({
                    url: url,
                    method: this.form.method,
                    port: this.form.port,
                    header: header,
                    agent: false
                });
                let result = [];
                let chunks = [];
                let size = 0;
                request.setHeader('referer', url);
                request.on('response', (response) => {
                    console.log(response.headers);
                    if (response.statusCode !== 200) {
                        this.loading = false;
                        this.$message.warning('请求错误');
                    }
                    response.on('data', (chunk) => {
                        console.log('获取data');
                        if(this.isCancel === true) {
                            request.destroy();
                            return false;
                        }
                        chunks.push(chunk);
                        size += chunk.length;
                        result.push(chunk);
                        if (this.timmer2) {
                            clearTimeout(this.timmer2);
                        }
                        this.timmer2 = setTimeout(() => {
                            let listReg = new RegExp(this.form.listStart + '([\\s\\S]*)' + this.form.listEnd + '', 'gi');
                            let imgReg = new RegExp(this.form.urlStart + '.*?(?:>|\\' + this.form.urlEnd + ')', 'gi');
                            let srcReg = /href=[\'\"]?([^\'\"]*)[\'\"]?/i;
                            let buf = myBuffer.concat(chunks, size);
                            let checkEncoding = true;
                            try {
                                this.listResult = iconv.decode(buf, this.form.encoding);
                            } catch (e) {
                                console.warn(e);
                                checkEncoding = false;
                            } finally {
                                if (!checkEncoding) {
                                    this.$message.warning('页面编码填写错误');
                                    return false;
                                }
                                let listBody = this.listResult.match(listReg);
                                if (listBody) {
                                    let arr = listBody[0].match(imgReg);
                                    if(arr && arr.length > 0) {
                                        arr.forEach((item) => {
                                            let src = item.match(srcReg);
                                            if (src.length > 0 && src[1]) {
                                                if (src[1].indexOf('https://') < 0 && src[1].indexOf('http://') < 0) {
                                                    src[1] = this.form.imgServ + src[1];
                                                }
                                                this.urlList.push({
                                                    url: src[1],
                                                });
                                            }
                                        });
                                        if (callback && typeof callback == 'function') {
                                            callback(true);
                                        }
                                    }
                                } else {
                                    this.$message.warning('部分列表获取失败，请检查过滤规则是否正确');
                                    if (callback && typeof callback == 'function') {
                                        callback(false);
                                    }
                                }
                            }
                        }, this.form.timeout);
                    });
                    response.on('end', () => {
                        console.log(`response end`)
                    });
                    response.on('error', (error) => {
                        console.log(`ERROR: ${JSON.stringify(error)}`)
                    })
                });
                request.end();
            },
            getList (callback) {

                let port = this.form.port;
                this.result = '';
                this.imgList = [];
                this.tempImgList = [];
                this.getImgListLoading = false;
                this.percentage = 0;
                let url = port !== 80 ? (this.form.url + ':' + port) : this.form.url;
                if (url.indexOf('https://') < 0 && url.indexOf('http://') < 0) {
                    url = 'https://' + url;
                }
                if (url.indexOf('(*)') > -1) {
                    console.log('检测到通配符');
                    let wildcardStart = this.form.wildcardStart;
                    let wildcardEnd = this.form.wildcardEnd;
                    if (wildcardStart > wildcardEnd) {
                        this.$alert('通配符结束必须大于开始', '通配符错误', {
                            confirmButtonText: '确定',
                            type: 'warning',
                            callback: action => {
                            }
                        });
                        return false;
                    } else {
                        // 单个规则测试只请求第一个页面
                        let list = [];
                        let arr = url.split('(*)');
                        for (let i = wildcardStart; i <= wildcardEnd; i++) {
                            let urlItem = arr[0] + i + arr[1];
                            list.push(urlItem);
                        }
                        console.log(list);
                        let loop = (i) => {
                            console.log('循环开始', i);
                            if (this.isCancel === true) {
                                return false;
                            }
                            if (i == 0) {
                                this.urlList = [];
                            }
                            this.getListByUrl(list[i],() => {
                                if (i >= list.length - 1) {
                                    this.$message.success('列表获取成功');
                                    this.loading = false;
                                    if (callback && typeof callback == 'function') {
                                        callback(false);
                                    }
                                } else {
                                    i = i + 1;
                                    this.getListLoadingText = `正在获取列表(${i}/${list.length})`;
                                    this.dialogText = `正在获取列表(${i}/${list.length})`;
                                    this.dialogPercentage = i / list.length * 100;
                                    loop(i);
                                }
                            });
                        };
                        loop(0);
                    }
                } else {
                    this.urlList = [];
                    this.getListByUrl(url, () => {
                        if (callback && typeof callback == 'function') {
                            callback(false);
                        }
                    });
                }
            },
            loadRuleFile() {
                let loadRuleFilePath = localStorage.getItem('loadRuleFilePath') ? localStorage.getItem('loadRuleFilePath') : os.homedir() + '\\Desktop';
                console.log(loadRuleFilePath);
                let file = electron.remote.dialog.showOpenDialogSync({
                    title: '选择规则文件',
                    buttonLabel: '载入',
                    defaultPath: loadRuleFilePath,
                    properties: ['openFile', 'promptToCreate'],
                    filters: [
                        {name: 'Json', extensions: ['json']},
                        {name: '所有文件', extensions: ['*']}
                    ]
                });
                if (file) {
                    console.log(file);
                    let saveJSON = fsExtra.readJsonSync(file[0]);
                    this.form = saveJSON;
                    this.result = '';
                    this.listResult = '';
                    this.imgSrcResult = '';
                    this.imgList = [];
                    this.tempImgList = [];
                    this.urlList = [];
                    let filePath = file[0];
                    let arr = filePath.split('\\');
                    let fileName = arr[arr.length - 1];
                    let path = filePath.replace(fileName, '');
                    console.log('filePath', filePath);
                    console.log('arr', arr);
                    console.log('fileName', fileName);
                    console.log('path', path);
                    localStorage.setItem('loadRuleFilePath', path);
                    this.$message.success('载入成功');
                }
            },
            saveRuleFile() {

                let defaultPath = localStorage.getItem('ruleSavePath') ? localStorage.getItem('ruleSavePath') : os.homedir() + '\\Desktop\\' + this.form.ruleName + '.json';
                let file = electron.remote.dialog.showSaveDialogSync({
                    title: '选择保存目录',
                    buttonLabel: '保存',
                    defaultPath: defaultPath,
                    properties: ['openFile', 'promptToCreate'],
                    filters: [
                        {name: 'Json', extensions: ['json']}
                    ]
                });
                if (file) {
                    let saveJSON = this.form;
                    fsExtra.outputJsonSync(file, saveJSON);
                    let filePath = file;
                    let arr = filePath.split('\\');
                    let fileName = arr[arr.length - 1];
                    let path = filePath.replace(fileName, '');
                    localStorage.setItem('ruleSavePath', path);
                    this.$message.success('保存成功');
                }
            },
            choosePath() {

                let path = electron.remote.dialog.showOpenDialogSync({
                    title: '选择保存目录',
                    defaultPath: this.defaultSaveImgPath,
                    properties: ['openDirectory']
                });
                if (path) {
                    this.defaultSaveImgPath = path[0];
                    localStorage.setItem('defaultSaveImgPath', path[0]);
                }
            },
            cancel() {
                this.isCancel = true;
                this.dialogVisible = false;
                this.$alert('任务已停止', '中止任务', {
                    confirmButtonText: '确定',
                    type: 'warning',
                    callback: action => {
                    }
                });
                setTimeout(() => {
                    this.isCancel = false;
                    this.loading = false;
                    this.getImgListLoading = false;
                }, 1000);
            },
            saveFile(path, file, callback) {
                let save = fs.writeFileSync(path, file);
                if (!save) {
                    console.log('保存成功');
                    callback(true);
                } else {
                    callback(false);
                    console.log('保存失败');
                }
            },
            getImg(src, callback) {

                console.log('请求图片', src);
                let header = {
                    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
                    "Accept-Encoding": "gzip, deflate, br",
                    "Accept-Language": "zh-CN,zh;q=0.9",
                    "Cache-Control": "no-cache",
                    "Connection": "keep-alive",
                    "referer": this.form.url,
                    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.105 Safari/537.36"
                };
                let chunks = [];
                let size = 0;
                const request = new electron.remote.net.request({
                    url: src,
                    method: 'GET',
                    header: header,
                    agent: false
                });
                // request.setHeader('referer',url);
                request.on('response', (response) => {
                    if (response.statusCode !== 200) {
                        this.$message.warning('请求错误');
                    }
                    let fileLength = response.headers['content-length'];
                    console.log('文件总长度：', fileLength);
                    response.on('data', (chunk) => {
                        if(this.isCancel === true) {
                            request.destroy();
                            return false;
                        }
                        size += chunk.length;
                        chunks.push(chunk);
                        this.percentage = (size / fileLength) * 100;
                        this.dialogPercentage = (size / fileLength) * 100;
                        if (size >= fileLength) {
                            let buf = myBuffer.concat(chunks, size);
                            let imgInfo = imgSize(buf);
                            // let fileName = new Date().getTime() + '.'+imgInfo.type;
                            let fileName = src.split('/')[src.split('/').length - 1];
                            this.percentage = 100;
                            this.saveFile(this.defaultSaveImgPath + '\\' + fileName, buf, (res) => {
                                if (res) {
                                    let base64Img = 'data:image/' + imgInfo.type + ';base64,';
                                    base64Img += buf.toString('base64');
                                    if (callback && typeof callback == 'function') {
                                        callback(base64Img);
                                    }
                                }
                            });
                        }
                    });
                    response.on('end', () => {
                        console.info('getImg end');
                        // let base64Img = 'data:image/jpg;base64,';
                        // let buf = myBuffer.concat(chunks,size);
                        // base64Img+=buf.toString('base64');
                        // if(callback && typeof callback == 'function') {
                        //     callback(base64Img);
                        // }
                    });
                    response.on('error', (error) => {
                        console.log(`ERROR: ${JSON.stringify(error)}`)
                        this.$alert('下载图片出错', '错误', {
                            confirmButtonText: '确定',
                            type: 'warning',
                            callback: action => {
                            }
                        });
                    })
                });
                request.end();
            },
            getImgList(list, callback) {
                console.log(list);

                this.imgList = [];
                this.tempImgList = [];
                this.getImgListLoading = true;
                this.getImgListLoadingText = '正在获取图片地址';
                let loop = (i) => {
                    if (this.isCancel === true) {
                        return false;
                    }
                    this.getUrl(list[i].url, () => {
                        if (i < list.length - 1) {
                            this.getImgListLoadingText = `正在获取图片地址(${i + 1}/${list.length})`;
                            this.dialogText = `正在获取图片地址(${i + 1}/${list.length})`;
                            this.dialogPercentage = i / list.length * 100;
                            i++;
                            loop(i);
                        } else {
                            this.getImgListLoading = false;
                            if (this.tempImgList.length > 0) {
                                this.$message.success('图片地址获取完成，等待下载');
                                if (callback && typeof callback == 'function') {
                                    callback(true);
                                }
                            } else {
                                this.$alert('没有获取到图片地址，请检查规则是否有误', '操作异常', {
                                    confirmButtonText: '确定',
                                    type: 'warning',
                                    callback: action => {
                                    }
                                });
                                if (callback && typeof callback == 'function') {
                                    callback(false);
                                }
                            }
                        }
                    });
                };
                loop(0);
            },
            getUrl(listUrl, callback) {

                let port = this.form.port;
                let url = port !== 80 ? (listUrl + ':' + port) : listUrl;
                if (url.indexOf('https://') < 0 && url.indexOf('http://') < 0) {
                    url = 'https://' + url;
                }
                let header = {
                    "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
                    "Accept-Encoding": "gzip, deflate, br",
                    "Accept-Language": "zh-CN,zh;q=0.9",
                    "Cache-Control": "no-cache",
                    "Connection": "keep-alive",
                    "referer": url,
                    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.105 Safari/537.36"
                };
                this.loading = true;
                const request = new electron.remote.net.request({
                    url: url,
                    method: this.form.method,
                    port: this.form.port,
                    header: header,
                    agent: false
                });
                let result = [];
                let chunks = [];
                let size = 0;
                request.setHeader('referer', url);
                request.on('response', (response) => {
                    console.log(response);
                    if (response.statusCode !== 200) {
                        this.$message.warning('请求错误');
                        this.getImgListLoading = false;
                    }
                    response.on('data', (chunk) => {
                        console.log('获取data');
                        if(this.isCancel === true) {
                            request.destroy();
                            return false;
                        }
                        chunks.push(chunk);
                        size += chunk.length;
                        result.push(chunk);
                        if (this.timmer2) {
                            clearTimeout(this.timmer2);
                        }
                        this.timmer2 = setTimeout(() => {
                            let imgReg = /<img.*?(?:>|\/>)/gi;
                            // let srcReg = /src=\"?([^"]*)\"\salt=\"Wallpaper\"?/i; // imgSrcStart //imgSrcEnd
                            let srcReg = new RegExp(this.form.imgSrcStart + '?([^"]*)\\"' + this.form.imgSrcEnd + '?', 'i');
                            console.log(srcReg);
                            let buf = myBuffer.concat(chunks, size);
                            let checkEncoding = true;
                            try {
                                this.imgSrcResult = iconv.decode(buf, this.form.encoding);
                            } catch (e) {
                                console.warn(e);
                                checkEncoding = false;
                            } finally {
                                if (!checkEncoding) {
                                    this.$message.warning('页面编码填写错误');
                                    clearTimeout(this.timmer2);
                                    return false;
                                }
                                if (srcReg) {
                                    let arr = this.imgSrcResult.match(imgReg);
                                    if (arr) {
                                        arr.forEach((item) => {
                                            let src = item.match(srcReg);
                                            if (src && src.length > 0 && src[1]) {
                                                if (src[1].indexOf('https://') < 0 && src[1].indexOf('http://') < 0) {
                                                    src[1] = this.form.imgServ + src[1];
                                                }
                                                this.tempImgList.push({
                                                    src: src[1],
                                                    isDownload: 1,
                                                });
                                            }
                                        });
                                        if (callback && typeof callback == 'function') {
                                            callback(true);
                                        }
                                    }
                                }
                                this.loading = false;
                            }
                        }, this.form.timeout);
                    });
                    response.on('error', (error) => {
                        console.log(`ERROR: ${JSON.stringify(error)}`)
                    })
                });
                request.end();
            },
            downloadPictureByList(list, callback) {

                this.loading = true;
                console.log(list.length + '张图片');
                let loop = (i) => {
                    if(this.isCancel === true) {
                        return false;
                    }
                    let src = list[i].src;
                    this.dialogText = `正在获取第${i}张图片(${i}/${list.length})`;
                    this.tempImgList[i].isDownload = 2;
                    this.getImg(src, (res) => {
                        this.imgList.push(res);
                        this.tempImgList[i].isDownload = 3;
                        if (i < list.length - 1) {
                            let n = i + 1;
                            loop(n);
                        } else {
                            this.loading = false;
                            this.percentage = 0;
                            if (callback && typeof callback == 'function') {
                                callback(true);
                            }
                            this.$alert('图片下载完成', '下载完成', {
                                confirmButtonText: '确定',
                                type: 'success',
                                callback: action => {
                                }
                            });
                        }
                    });
                };
                loop(0);
            },
            progressText(percentage) {
                if (percentage == 0) {
                    return '等待下载'
                }
                if (percentage > 0 && percentage < 100) {
                    return '下载中'
                }
                if (percentage > 100) {
                    return '已完成'
                }
            },
            runRuleFile() {

                this.$confirm('将自动执行当前规则，请确定规则无误', '提示', {
                    type: 'warning',
                    confirmButtonText: '执行',
                    cancelButtonText: '取消',
                }).then(() => {
                    this.dialogVisible = true;
                    this.dialogPercentage = 0;
                    this.dialogText = 'Starting...';
                    this.getList(() => {
                        this.getImgList(this.urlList, () => {
                            this.downloadPictureByList(this.tempImgList, () => {
                                this.dialogVisible = false;
                                this.$alert('已完成下载任务', '任务结束', {
                                    confirmButtonText: '确定',
                                    type: 'warning',
                                    callback: action => {
                                    }
                                });
                            });
                        });
                    });
                }).catch(() => {
                });
            },
        },
    }
</script>
<style lang="scss" scoped>
    .spiderBox {
        overflow-x: hidden;
        overflow-y: scroll;
        height: 500px;
    }

    .downloadImg {
        width: 100px;
        height: 100px;
        margin: 0;
        display: inline-block;
        vertical-align: top;
    }

    .downloadImgPlaceholder {
        display: inline-block;
        vertical-align: top;
        width: 100px;
        height: 100px;
        padding: 10px;
        box-sizing: border-box;
        margin: 0;
        background: #f5f5f5;
        border-radius: 0px;
    }
</style>
