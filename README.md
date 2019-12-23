import axios from 'axios';
import openSocket from "socket.io-client";
const api = "http://35.154.54.127:3068";
const socket = openSocket("http://35.154.54.127:7896", {transports: ['websocket', 'polling', 'flashsocket'] });

let socketId;
socket.on('connect',()=>{
    socketId = socket.id
    console.log("Heres socketId ",socketId)
})
const apiKey = "BNBUQUTPSYH"
const externalFunctions = {
/**
 * Returns API Key for vendor
 * @returns {String} An APIKey wor the Web
 */
    async getApiKey(){
        return apiKey
    },    
 /**
  *
  *
  * @returns {string } qrdata string with 
  */
 async  generateqr() {
        return new Promise((resolve, reject) => {
            console.log("apiKey ",apiKey)
            try{
            if(!apiKey){
                return new Error("apiKey not availaible")
            }
            if(!socketId){
                console.log("socket not availaible try to reload page")
                return new Error("socketId not availaible")
            }
            console.log("request Payload ",socketId)
            //get session key from server and store the socketID with session key in db , to be used further during service Provider API to emit
            axios.get(`${api}/authorize/generateqr?apiKey=${apiKey}&socketId=${socketId}`, { crossdomain: true } )
                .then(response => {
                    if(response.data.code === 400){
                        let errorString = response.data.message? response.data.message:"Something went wrong"
                        return reject(errorString);
                    }   
                    console.log("Secret key ", response)
                    // return resolve(response.data.result.toString());
                    let qrData =`{"apikey":"${apiKey}","encryptionkey":"1234567","reqNo":"${response.data.result}","sessionKey":"${response.data.result}" }`                    
                    return resolve(qrData);
                })
                .catch(e => {
                    console.log("This is e ", e)
                    return reject("unable to get secret Token")
                })
            }catch(e){
                console.log("Error in qr Code ",e)
                return reject(e) 
            }
        })
    },
/**
 *Listen for the Service provider response
 *
 * @param {*} cb Gives a call ack with encapsulated  data
 */
async  listenForServiceProviderResponse(cb) {
        console.log("listenForServiceProviderResponse   ")
        socket.on(`sendServiceProvider`, data => {
            console.log("Got Data ",data)
            cb(null, data);
        });
    },
/**
 *A Socket Listener for user data will return data if user authorizes the request else will return false
 *
 * @param {*} cb
 */
async listenForUserData(cb) {
        console.log("listenForServiceProviderResponse   ")
        socket.on(`userdata`, data => {
            console.log("Got Data userdata = = =",data)
            cb (null,data);
        });
    }
}
export default externalFunctions
