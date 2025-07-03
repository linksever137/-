// Cloudflare Worker - 随机身份信息生成器

// 姓氏列表
const surnames = ['王', '李', '张', '刘', '陈', '杨', '赵', '黄', '周', '吴', '徐', '孙', '胡', '朱', '高', '林', '何', '郭', '马', '罗', '梁', '宋', '郑', '谢', '韩', '唐', '冯', '于', '董', '萧', '程', '曹', '袁', '邓', '许', '傅', '沈', '曾', '彭', '吕'];

// 名字列表
const maleNames = ['伟', '强', '磊', '洋', '勇', '军', '杰', '涛', '明', '刚', '平', '辉', '鹏', '华', '飞', '斌', '志', '宇', '浩', '凯', '健', '俊', '帅', '威', '龙', '翔', '波', '成', '进', '峰'];
const femaleNames = ['芳', '燕', '敏', '艳', '丽', '娟', '静', '秀', '娜', '梅', '洁', '琳', '玲', '颖', '红', '娇', '凤', '莉', '婷', '慧', '巧', '美', '媛', '瑶', '雪', '倩', '珍', '妍', '晶', '华'];

// 省份和城市数据
const provinces = {
  '11': { name: '北京市', cities: ['东城区', '西城区', '朝阳区', '海淀区', '丰台区', '石景山区'] },
  '12': { name: '天津市', cities: ['和平区', '河东区', '河西区', '南开区', '河北区', '红桥区'] },
  '13': { name: '河北省', cities: ['石家庄市', '唐山市', '秦皇岛市', '邯郸市', '邢台市', '保定市'] },
  '14': { name: '山西省', cities: ['太原市', '大同市', '阳泉市', '长治市', '晋城市', '朔州市'] },
  '21': { name: '辽宁省', cities: ['沈阳市', '大连市', '鞍山市', '抚顺市', '本溪市', '丹东市'] },
  '31': { name: '上海市', cities: ['黄浦区', '徐汇区', '长宁区', '静安区', '普陀区', '虹口区'] },
  '32': { name: '江苏省', cities: ['南京市', '无锡市', '徐州市', '常州市', '苏州市', '南通市'] },
  '33': { name: '浙江省', cities: ['杭州市', '宁波市', '温州市', '嘉兴市', '湖州市', '绍兴市'] },
  '44': { name: '广东省', cities: ['广州市', '深圳市', '珠海市', '汕头市', '佛山市', '韶关市'] },
  '50': { name: '重庆市', cities: ['万州区', '涪陵区', '渝中区', '大渡口区', '江北区', '沙坪坝区'] }
};

// 街道名称
const streets = ['中山路', '解放路', '建设路', '人民路', '和平路', '北京路', '南京路', '长江路', '黄河路', '文化路', '青年路', '光明路', '新华路', '民主路', '自由路', '幸福路', '富强路', '团结路', '胜利路', '友谊路'];

// 公司名称词汇
const companyPrefixes = ['华', '鑫', '诚', '信', '盛', '泰', '恒', '瑞', '金', '银', '通', '达', '创', '新', '智', '慧', '东', '方', '天', '地'];
const companySuffixes = ['科技', '网络', '信息', '软件', '电子', '数码', '通信', '贸易', '商贸', '实业', '投资', '咨询', '服务', '传媒', '文化'];

// 职业列表
const occupations = ['软件工程师', '产品经理', '设计师', '销售经理', '市场专员', '人力资源', '财务会计', '行政助理', '教师', '医生', '护士', '律师', '建筑师', '工程师', '记者', '编辑', '摄影师', '厨师', '司机', '客服专员'];

// 邮箱域名
const emailDomains = ['qq.com', '163.com', '126.com', 'gmail.com', 'outlook.com', 'hotmail.com', 'sina.com', 'sohu.com', '139.com', 'yeah.net'];

// 银行列表
const banks = ['中国工商银行', '中国建设银行', '中国银行', '中国农业银行', '交通银行', '招商银行', '浦发银行', '中信银行', '光大银行', '民生银行'];

// 生成随机数
function random(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

// 生成随机日期
function randomDate(minYear, maxYear) {
  const year = random(minYear, maxYear);
  const month = random(1, 12);
  const day = random(1, 28); // 简化处理，避免月份日期问题
  return { year, month, day };
}

// 计算身份证校验码
function calculateChecksum(idNumber) {
  const weights = [7, 9, 10, 5, 8, 4, 2, 1, 6, 3, 7, 9, 10, 5, 8, 4, 2];
  const checksums = ['1', '0', 'X', '9', '8', '7', '6', '5', '4', '3', '2'];
  
  let sum = 0;
  for (let i = 0; i < 17; i++) {
    sum += parseInt(idNumber[i]) * weights[i];
  }
  
  return checksums[sum % 11];
}

// 生成身份证号
function generateIdNumber(provinceCode, birthDate, gender) {
  const year = birthDate.year.toString();
  const month = birthDate.month.toString().padStart(2, '0');
  const day = birthDate.day.toString().padStart(2, '0');
  
  // 生成区县代码（简化处理）
  const countyCode = random(1, 20).toString().padStart(2, '0');
  
  // 生成顺序码（第17位奇数为男性，偶数为女性）
  const sequenceCode = random(0, 99) * 10 + (gender === '男' ? 1 : 0);
  const sequence = sequenceCode.toString().padStart(3, '0');
  
  const idWithoutChecksum = provinceCode + countyCode + year + month + day + sequence;
  const checksum = calculateChecksum(idWithoutChecksum);
  
  return idWithoutChecksum + checksum;
}

// 生成手机号
function generatePhoneNumber() {
  const prefixes = ['130', '131', '132', '133', '134', '135', '136', '137', '138', '139', '150', '151', '152', '153', '155', '156', '157', '158', '159', '170', '176', '177', '178', '180', '181', '182', '183', '184', '185', '186', '187', '188', '189'];
  const prefix = prefixes[random(0, prefixes.length - 1)];
  const suffix = random(10000000, 99999999);
  return prefix + suffix;
}

// 生成银行卡号（简化版）
function generateBankCardNumber() {
  // 使用常见的银行卡号前缀
  const prefixes = ['6222', '6225', '6228', '6229', '5187', '4563', '4033', '4062'];
  const prefix = prefixes[random(0, prefixes.length - 1)];
  const middle = random(100000000000, 999999999999).toString();
  return prefix + middle;
}

// 生成完整的身份信息
function generateIdentity() {
  // 基本信息
  const gender = random(0, 1) === 0 ? '男' : '女';
  const surname = surnames[random(0, surnames.length - 1)];
  const nameList = gender === '男' ? maleNames : femaleNames;
  const name = surname + nameList[random(0, nameList.length - 1)];
  
  // 生日和年龄
  const currentYear = new Date().getFullYear();
  const birthDate = randomDate(currentYear - 60, currentYear - 18);
  const age = currentYear - birthDate.year;
  
  // 地址信息
  const provinceKeys = Object.keys(provinces);
  const provinceCode = provinceKeys[random(0, provinceKeys.length - 1)];
  const province = provinces[provinceCode];
  const city = province.cities[random(0, province.cities.length - 1)];
  const street = streets[random(0, streets.length - 1)];
  const streetNumber = random(1, 999);
  const building = random(1, 50);
  const unit = random(1, 6);
  const room = random(101, 3201);
  const address = `${province.name}${city}${street}${streetNumber}号${building}栋${unit}单元${room}室`;
  
  // 身份证号
  const idNumber = generateIdNumber(provinceCode, birthDate, gender);
  
  // 联系方式
  const phoneNumber = generatePhoneNumber();
  const email = name.toLowerCase() + random(1000, 9999) + '@' + emailDomains[random(0, emailDomains.length - 1)];
  
  // 工作信息
  const occupation = occupations[random(0, occupations.length - 1)];
  const companyName = companyPrefixes[random(0, companyPrefixes.length - 1)] + 
                      companyPrefixes[random(0, companyPrefixes.length - 1)] + 
                      companySuffixes[random(0, companySuffixes.length - 1)] + 
                      '有限公司';
  const monthSalary = random(3, 50) * 1000;
  
  // 银行信息
  const bank = banks[random(0, banks.length - 1)];
  const bankCardNumber = generateBankCardNumber();
  
  // 其他信息
  const height = gender === '男' ? random(165, 185) : random(155, 175);
  const weight = gender === '男' ? random(60, 90) : random(45, 70);
  const bloodTypes = ['A', 'B', 'AB', 'O'];
  const bloodType = bloodTypes[random(0, 3)] + (random(0, 1) === 0 ? '+' : '-');
  const maritalStatus = age > 25 && random(0, 1) === 0 ? '已婚' : '未婚';
  const education = ['高中', '大专', '本科', '硕士', '博士'][random(0, 4)];
  
  return {
    // 基本信息
    name,
    gender,
    age,
    birthDate: `${birthDate.year}年${birthDate.month}月${birthDate.day}日`,
    idNumber,
    
    // 联系方式
    phoneNumber,
    email,
    
    // 地址信息
    province: province.name,
    city,
    address,
    
    // 工作信息
    occupation,
    company: companyName,
    monthSalary: monthSalary + '元',
    
    // 银行信息
    bank,
    bankCardNumber,
    
    // 其他信息
    height: height + 'cm',
    weight: weight + 'kg',
    bloodType,
    maritalStatus,
    education,
    nationality: '中国',
    ethnicity: '汉族'
  };
}

// HTML模板
const htmlTemplate = `
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>随机身份信息生成器</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }
        
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }
        
        .container {
            background: rgba(255, 255, 255, 0.95);
            border-radius: 20px;
            box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
            max-width: 800px;
            width: 100%;
            padding: 40px;
            animation: fadeIn 0.5s ease-out;
        }
        
        @keyframes fadeIn {
            from {
                opacity: 0;
                transform: translateY(20px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }
        
        h1 {
            text-align: center;
            color: #5a67d8;
            margin-bottom: 30px;
            font-size: 2.5em;
            text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
        }
        
        .info-grid {
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
            gap: 20px;
            margin-bottom: 30px;
        }
        
        .info-section {
            background: #f7fafc;
            padding: 20px;
            border-radius: 10px;
            border: 1px solid #e2e8f0;
            transition: all 0.3s ease;
        }
        
        .info-section:hover {
            transform: translateY(-2px);
            box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);
        }
        
        .section-title {
            font-size: 1.2em;
            font-weight: bold;
            color: #4a5568;
            margin-bottom: 15px;
            display: flex;
            align-items: center;
            gap: 8px;
        }
        
        .info-item {
            margin-bottom: 10px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 5px 0;
            border-bottom: 1px solid #e2e8f0;
        }
        
        .info-item:last-child {
            border-bottom: none;
            margin-bottom: 0;
        }
        
        .label {
            font-weight: 500;
            color: #718096;
            font-size: 0.9em;
        }
        
        .value {
            color: #2d3748;
            font-weight: 400;
            text-align: right;
            word-break: break-all;
        }
        
        .generate-btn {
            display: block;
            width: 100%;
            padding: 15px 30px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            border-radius: 10px;
            font-size: 1.1em;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.3s ease;
            box-shadow: 0 4px 15px rgba(102, 126, 234, 0.4);
        }
        
        .generate-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 6px 20px rgba(102, 126, 234, 0.6);
        }
        
        .generate-btn:active {
            transform: translateY(0);
        }
        
        .loading {
            display: none;
            text-align: center;
            padding: 20px;
            color: #718096;
        }
        
        .spinner {
            display: inline-block;
            width: 20px;
            height: 20px;
            border: 3px solid #f3f4f6;
            border-top: 3px solid #667eea;
            border-radius: 50%;
            animation: spin 1s linear infinite;
        }
        
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
        
        @media (max-width: 600px) {
            .container {
                padding: 20px;
            }
            
            h1 {
                font-size: 1.8em;
            }
            
            .info-grid {
                grid-template-columns: 1fr;
            }
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>🎲 随机身份信息生成器</h1>
        
        <div class="loading" id="loading">
            <div class="spinner"></div>
            <p>正在生成身份信息...</p>
        </div>
        
        <div id="result" style="display: none;">
            <div class="info-grid">
                <div class="info-section">
                    <h3 class="section-title">👤 基本信息</h3>
                    <div class="info-item">
                        <span class="label">姓名</span>
                        <span class="value" id="name">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">性别</span>
                        <span class="value" id="gender">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">年龄</span>
                        <span class="value" id="age">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">生日</span>
                        <span class="value" id="birthDate">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">身份证号</span>
                        <span class="value" id="idNumber" style="font-family: monospace;">-</span>
                    </div>
                </div>
                
                <div class="info-section">
                    <h3 class="section-title">📞 联系方式</h3>
                    <div class="info-item">
                        <span class="label">手机号</span>
                        <span class="value" id="phoneNumber">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">邮箱</span>
                        <span class="value" id="email">-</span>
                    </div>
                </div>
                
                <div class="info-section">
                    <h3 class="section-title">🏠 地址信息</h3>
                    <div class="info-item">
                        <span class="label">省份</span>
                        <span class="value" id="province">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">城市</span>
                        <span class="value" id="city">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">详细地址</span>
                        <span class="value" id="address">-</span>
                    </div>
                </div>
                
                <div class="info-section">
                    <h3 class="section-title">💼 工作信息</h3>
                    <div class="info-item">
                        <span class="label">职业</span>
                        <span class="value" id="occupation">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">公司</span>
                        <span class="value" id="company">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">月薪</span>
                        <span class="value" id="monthSalary">-</span>
                    </div>
                </div>
                
                <div class="info-section">
                    <h3 class="section-title">💳 银行信息</h3>
                    <div class="info-item">
                        <span class="label">开户行</span>
                        <span class="value" id="bank">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">银行卡号</span>
                        <span class="value" id="bankCardNumber" style="font-family: monospace;">-</span>
                    </div>
                </div>
                
                <div class="info-section">
                    <h3 class="section-title">📋 其他信息</h3>
                    <div class="info-item">
                        <span class="label">身高</span>
                        <span class="value" id="height">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">体重</span>
                        <span class="value" id="weight">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">血型</span>
                        <span class="value" id="bloodType">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">婚姻状况</span>
                        <span class="value" id="maritalStatus">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">学历</span>
                        <span class="value" id="education">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">国籍</span>
                        <span class="value" id="nationality">-</span>
                    </div>
                    <div class="info-item">
                        <span class="label">民族</span>
                        <span class="value" id="ethnicity">-</span>
                    </div>
                </div>
            </div>
        </div>
        
        <button class="generate-btn" onclick="generateNewIdentity()">生成新身份信息</button>
    </div>
    
    <script>
        async function generateNewIdentity() {
            const loading = document.getElementById('loading');
            const result = document.getElementById('result');
            
            loading.style.display = 'block';
            result.style.display = 'none';
            
            try {
                const response = await fetch('/api/generate');
                const data = await response.json();
                
                // 更新显示的信息
                for (const [key, value] of Object.entries(data)) {
                    const element = document.getElementById(key);
                    if (element) {
                        element.textContent = value;
                    }
                }
                
                loading.style.display = 'none';
                result.style.display = 'block';
            } catch (error) {
                console.error('生成失败:', error);
                loading.style.display = 'none';
                alert('生成失败，请重试');
            }
        }
        
        // 页面加载时自动生成一次
        window.addEventListener('load', generateNewIdentity);
    </script>
</body>
</html>
`;

// Worker处理函数
async function handleRequest(request) {
  const url = new URL(request.url);
  
  if (url.pathname === '/api/generate') {
    // 生成身份信息API
    const identity = generateIdentity();
    return new Response(JSON.stringify(identity), {
      headers: {
        'Content-Type': 'application/json',
        'Access-Control-Allow-Origin': '*'
      }
    });
  }
  
  // 返回HTML页面
  return new Response(htmlTemplate, {
    headers: {
      'Content-Type': 'text/html; charset=utf-8'
    }
  });
}

// 导出Worker处理函数
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request));
});
