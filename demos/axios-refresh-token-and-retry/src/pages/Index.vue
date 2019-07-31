<template>
  <q-page class="flex flex-center">
    <img alt="Quasar logo" src="~assets/quasar-logo-full.svg">
  </q-page>
</template>

<style>
</style>

<script>
import axios from 'axios'
const customAxios = axios.create()
customAxios.interceptors.response.use(
  function (response) {
    return response
  },
  function (error) {
    const errorResponse = error.response
    if (errorResponse.status === 401) {
      return resetToken(error)
    }
    return Promise.reject(error)
  }
)

let isAlreadyFetchingAccessToken = false
let subscribers = []

function getResetToken () {
  return 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MTksImNvbXBhbnlJZCI6NywiaXNBZG1pbiI6dHJ1ZSwiZmlyc3ROYW1lIjoiQWFyb24iLCJsYXN0TmFtZSI6Ikd1byIsImF2YXRhciI6bnVsbCwiaWF0IjoxNTY0NTM2MTMwLCJleHAiOjE1NjUxNDA5MzB9.jt2vdl3plHwda0Tk9oa25KSM3a_6uckqgjAQ_zFHMl4'
}

async function resetToken (error) {
  try {
    const { response: errorResponse } = error
    const resetToken = await getResetToken()
    if (!resetToken) {
      return Promise.reject(error)
    }

    const retryOriginalRequest = new Promise(resolve => {
      addSubscriber(accessToken => {
        errorResponse.config.headers.Authorization = 'Bearer ' + accessToken
        resolve(axios(errorResponse.config))
      })
    })

    if (!isAlreadyFetchingAccessToken) {
      isAlreadyFetchingAccessToken = true
      const response = await axios({
        method: 'post',
        url: 'http://localhost:5000/api/v1/session/token',
        headers: {
          'Authorization': 'Bearer ' + resetToken
        },
        data: {
          id: 19
        }
      })

      if (!response.data) {
        return Promise.reject(error)
      }

      const newToken = response.data.token
      isAlreadyFetchingAccessToken = false
      onAccessTokenFetched(newToken)
    }
    return retryOriginalRequest
  } catch (err) {
    return Promise.reject(err)
  }
}

function onAccessTokenFetched (accessToken) {
  subscribers.forEach(callback => callback(accessToken))
  subscribers = []
}

function addSubscriber (callback) {
  subscribers.push(callback)
}

export default {
  name: 'PageIndex',
  async created () {
    customAxios.get('http://localhost:5000/api/v1/users')
      .then(result => {
        console.log('>', result)
      })

    // console.log('>', res)
    // alert(1)
    // const refreshAuthLogic = failedRequest => {
    //   alert(0)
    //   return axios({
    //     url: 'http://localhost:5000/api/v1/session/token',
    //     headers: {
    //       'Authorization': `Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MTksImNvbXBhbnlJZCI6NywiaXNBZG1pbiI6dHJ1ZSwiZmlyc3ROYW1lIjoiQWFyb24iLCJsYXN0TmFtZSI6Ikd1byIsImF2YXRhciI6bnVsbCwiaWF0IjoxNTY0NTM2MTMwLCJleHAiOjE1NjUxNDA5MzB9.jt2vdl3plHwda0Tk9oa25KSM3a_6uckqgjAQ_zFHMl4`
    //     }
    //   })
    // }
    // // .then(tokenRefreshResponse => {
    // //   alert(3)
    // //   localStorage.setItem('token', tokenRefreshResponse.data.token)
    // //   failedRequest.response.config.headers['Authentication'] = 'Bearer ' + tokenRefreshResponse.data.token
    // //   return Promise.resolve()
    // // })

    // // Instantiate the interceptor (you can chain it as it returns the axios instance)
    // // createAuthRefreshInterceptor(axios, refreshAuthLogic)

    // // Make a call. If it returns a 401 error, the refreshAuthLogic will be run,
    // // and the request retried with the new token
    // alert(2)
    // createAuthRefreshInterceptor(axios, refreshAuthLogic).get('http://localhost:5000/api/v1/users')
    //   .then(/* ... */)
    //   .catch(err => {
    //     alert(4)
    //     console.log(err)
    //   })
  }
}
</script>
