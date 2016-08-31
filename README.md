# WebApiusing EntityLibrary;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Net;
using System.Net.Http;
using System.Web.Http;

namespace MLSWebAPI.Controllers
{
    [Authorize]
    public class MLSController : ApiController
    {

        public HttpResponseMessage GetAllUserCodes(string userName)
        {

            try
            {
                var AList = EntityLibrary.ActivationCodes.GetAllUserCodes(UserName: userName);
                if(AList.Count()==0)
                {
                    return Request.CreateResponse(HttpStatusCode.NotFound,"No Activation Codes for the given UserName");
                }
                else
                {
                    return Request.CreateResponse<IEnumerable<ActivationCodes>>(HttpStatusCode.OK, AList);
                }
              
            }
            catch (Exception e)
            {
                HttpResponseMessage objResponse = new HttpResponseMessage(statusCode:HttpStatusCode.BadGateway);
                objResponse.ReasonPhrase = e.Message;
                throw new HttpResponseException(objResponse);
            }
        }
        [Route("api/MLS/GetAllActiveProducts")]
        public HttpResponseMessage GetAllActiveProducts(int active, string userName)
        {

            try
            {
                var AList = EntityLibrary.ActivationCodeProducts.GetActiveProductCodes(Active: active, UserName: userName);
                if (AList.Count() == 0)
                {
                    return Request.CreateResponse(HttpStatusCode.NotFound, "No Active codeproducts for the given UserName");
                }
                else
                {
                    return Request.CreateResponse<IEnumerable<ActivationCodeProducts>>(HttpStatusCode.OK, AList);
                }

                
            }
            catch (Exception e)
            {
                
                HttpResponseMessage objResponse = new HttpResponseMessage(statusCode:HttpStatusCode.BadGateway);
                objResponse.ReasonPhrase = e.Message;
                throw new HttpResponseException(objResponse);
            }
        }
        [Route("api/MLS/GetAllCourseProducts")]
        public HttpResponseMessage GetAllCourseProducts(int active, string userName)
        {
            try
            {
                var AList = EntityLibrary.CourseProducts.GetActiveList(Active: active, UserName: userName);
                if(AList.Count()==0)
                {
                    return Request.CreateResponse(HttpStatusCode.NotFound, "No Active courseproducts for the given UserName");
                }
                else
                {
                    return Request.CreateResponse<IEnumerable<CourseProducts>>(HttpStatusCode.OK, AList);
                }
            }
            catch (Exception e)
            {
                
                HttpResponseMessage objResponse = new HttpResponseMessage(statusCode:HttpStatusCode.BadGateway);
                objResponse.ReasonPhrase = e.Message;
                throw new HttpResponseException(objResponse);
            }
        }
        [Route("api/MLS/GetByControlCode")]
        public HttpResponseMessage GetByControlCode(string controlCode)
        {
            try
            {
                var AList = EntityLibrary.ActivationCodes.GetByControlCode(ControlCode: controlCode);

                if(AList.Count==0)
                {
                    return Request.CreateResponse(HttpStatusCode.NotFound, "No Active courseproducts for the given UserName");
                }
                
                else
                {
                      return Request.CreateResponse<IEnumerable<ActivationCodes>>(HttpStatusCode.OK, AList);
                }
             
            }
            catch (Exception e)
            {
                
                HttpResponseMessage objResponse = new HttpResponseMessage(statusCode:HttpStatusCode.BadGateway);
                objResponse.ReasonPhrase = e.Message;
                throw new HttpResponseException(objResponse);
            }
        }
        [Route("api/MLS/GetByActivationCodeID")]
        public HttpResponseMessage GetByActivationCodeID(int activationCodeID)
        {
            try
            {
                EntityLibrary.ActivationCodes objAC = new EntityLibrary.ActivationCodes(activationCodeID);
                if(objAC !=null)
                {
                    return Request.CreateResponse<ActivationCodes>(HttpStatusCode.OK, objAC);
                }
                else
                {
                      return Request.CreateResponse(HttpStatusCode.NotFound, "No Active codes for the given ActivationCodes");
                }
                
                
            }
            catch (Exception e)
            {

                HttpResponseMessage objResponse = new HttpResponseMessage(statusCode: HttpStatusCode.BadGateway);
                objResponse.ReasonPhrase = e.Message;
                throw new HttpResponseException(objResponse);
            }
        }
        [Route("api/MLS/SaveActivationCode")]
        public HttpResponseMessage PutSaveActivationCode(ActivationCodes objAC)
        {
            try
            {
                if (objAC != null)
                {
                    objAC.Save();
                    GlobalFunctions.SaveHistory(objAC.ActivationCodeID, (Int32)EntityLibrary.Enumerators.ActivationCodeHistoryType.Registered, 0, "Code is Registered", objAC.RegisteredBy);
                    return Request.CreateResponse(HttpStatusCode.OK, "ActivationCode table got updated");
                }
                else
                {
                    return Request.CreateResponse(HttpStatusCode.BadRequest, "Activation code object is null");
                }
            }
            catch (Exception e)
            {
                
                HttpResponseMessage objResponse = new HttpResponseMessage(statusCode: HttpStatusCode.BadGateway);
                objResponse.ReasonPhrase = e.Message;
                throw new HttpResponseException(objResponse);
            }
        }
        
    }
}
Call APi

using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Net.Http;
using Newtonsoft.Json;
using MLSWebAPI.Models;
using System.Threading.Tasks;
using EntityLibrary;


using System.Web.Configuration;
using System.Web.Script.Serialization;
using System.Text;
namespace MLS
{
    public class ServiceCalls
    {
        public string baseAddress = "";
        public string serviceAddress = "";

        public ServiceCalls()
        {
            baseAddress=WebConfigurationManager.AppSettings["baseAddress"];
            serviceAddress = WebConfigurationManager.AppSettings["serviceAddress"];
        }

        public async Task<ActivationCodes[]> GetAllUserCodes(string userName)
        {
            

            var client = new HttpClient();
            client.BaseAddress = new Uri(baseAddress);
            HttpResponseMessage response = await client.GetAsync(serviceAddress + "?apiKey=445-65-1216" + "&userName=" + userName);

            var msg = response.Content.ReadAsStringAsync().Result;
            ActivationCodes[] codes = JsonConvert.DeserializeObject<ActivationCodes[]>(msg);
            return codes;

        }

        public async Task<ActivationCodeProducts[]> GetAllActiveProducts(int active,string userName)
        {
            var client = new HttpClient();
            client.BaseAddress = new Uri(baseAddress);
            HttpResponseMessage response = await client.GetAsync(serviceAddress + "/GetAllActiveProducts" + "?apiKey=445-65-1216" + "&active=" + active +"&userName="+userName);

            var msg = response.Content.ReadAsStringAsync().Result;
            ActivationCodeProducts[] codes = JsonConvert.DeserializeObject<ActivationCodeProducts[]>(msg);
            return codes;
        }

        public async Task<CourseProducts[]> GetAllCourseProducts(int active, string userName)
        {

            var client = new HttpClient();
            client.BaseAddress = new Uri(baseAddress);
            HttpResponseMessage response = await client.GetAsync(serviceAddress + "/GetAllCourseProducts" + "?apiKey=445-65-1216" + "&active=" + active+"&userName="+userName);

            var msg = response.Content.ReadAsStringAsync().Result;
            CourseProducts[] codes = JsonConvert.DeserializeObject<CourseProducts[]>(msg);
            return codes;

        }

        public async Task<ActivationCodes[]> GetByControlCode(string controlCode)
        {
            var client = new HttpClient();
            client.BaseAddress = new Uri(baseAddress);
            HttpResponseMessage response = await client.GetAsync(serviceAddress + "/GetByControlCode" + "?apiKey=445-65-1216" + "&controlCode=" + controlCode);

            var msg = response.Content.ReadAsStringAsync().Result;
            ActivationCodes[] codes = JsonConvert.DeserializeObject<ActivationCodes[]>(msg);
            return codes;

        }

        public async Task<ActivationCodes> GetByActivationCodeID(int activationCodeID)
        {
            var client = new HttpClient();
            client.BaseAddress = new Uri(baseAddress);
            HttpResponseMessage response = await client.GetAsync(serviceAddress + "/GetByActivationCodeID" + "?apiKey=445-65-1216" + "&activationCodeID=" + activationCodeID);

            var msg = response.Content.ReadAsStringAsync().Result;
            ActivationCodes code = JsonConvert.DeserializeObject<ActivationCodes>(msg);
            return code;

        }

        public async void SaveActivationCode(ActivationCodes objAC)
        {
            var client = new HttpClient();
            client.BaseAddress = new Uri(baseAddress);
            HttpResponseMessage response = await client.PutAsJsonAsync(serviceAddress + "/SaveActivationCode" + "?apiKey=445-65-1216", objAC);

        }

        public async void PutActivateCode()
        {
            var baseAddress1 = baseAddress;
            var serviceAddress = "api/Test?apiKey=445-65-1216";
            var client = new HttpClient();
            client.BaseAddress = new Uri(baseAddress1);
            var values = new List<KeyValuePair<string, string>>();
            values.Add(new KeyValuePair<string, string>("ActivationCodeID", "1"));
            var content = new FormUrlEncodedContent(values);
            //StringContent q=new StringContent(data);
            HttpResponseMessage response = await client.PutAsync(serviceAddress, content);
        }


        
    }
}
