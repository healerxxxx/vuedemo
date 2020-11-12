import axios from 'axios'

/**
 * @description url参数转换
 * @date 2018-12-05
 * @param {String} temp 对应的url
 * @param {Object} data 要替换的参数
 * @return {String} 被替换完的url
 */
const template = (temp, data) => {
  return data ? temp.replace(/\{\{([^}]+)\}\}/g, function ($1, $2) {
    return data[$2] !== null || data[$2] !== undefined ? data[$2] : $1
  }) : temp
}

/**
 * @description 封装axios
 * @date 2018-12-05
 * @param {String} url 对应的 urls 中的key
 * @param {Object} param 请求的配置
 * @returns 一个Promise
 */
export default (url, param) => {
  if (!url) {
    return Promise.reject(new Error('URL路径错误'))
  }
  param = !param ? {} : param
  param.method = param.method || 'get'
  param.url = url
  if (param.urlParams) {
    param.url = template(param.url, param.urlParams)
  }
  param.headers = param.headers || {}
  /*
   * 设置数据格式
   */
  axios.defaults.headers.post['Content-Type'] = param.headers['Content-Type'] || 'application/x-www-form-urlencoded;charset=UTF-8'
  /*
   * 数据格式转换
   */
  param['transformRequest'] = function (data) {
    if (!data || typeof data !== 'object' || param.headers['Content-Type']) {
      return data
    }
    let arr = []
    for (let key in data) {
      arr.push(key + '=' + encodeURIComponent(typeof data[key] === 'object' ? JSON.stringify(data[key]) : data[key]))
    }
    return arr.join('&')
  }
  // 设置超时120秒
  axios.defaults.timeout = 120000
  // Promise返回axios获取的数据对象
  return axios(param).then((resp) => {
    if (resp.status && resp.status === 200) {
      return Promise.resolve(resp.data)
    } else {
      return Promise.reject(resp)
    }
  }, err => {
    let _err = null
    if (err.response) {
      _err = {
        status: err.response.status,
        statusText: err.response.statusText,
        url: url
      }
    } else {
      _err = `${url} 接口权限不足`
    }
    return Promise.reject(_err)
  })
}
