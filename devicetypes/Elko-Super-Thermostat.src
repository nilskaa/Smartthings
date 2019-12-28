/**
 *  Copyright 2015 SmartThings
 *
 *  Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except
 *  in compliance with the License. You may obtain a copy of the License at:
 *
 *      http://www.apache.org/licenses/LICENSE-2.0
 *
 *  Unless required by applicable law or agreed to in writing, software distributed under the License is distributed
 *  on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License
 *  for the specific language governing permissions and limitations under the License.
 *
 *	Elko ESH Plus Super TR RF
 *
 *	Authors: nilskaa, golfster187, RBoy Apps and torandreroland
 *	Date: 2019-11-25
 */
metadata {
	definition (name: "Elko Super Thermostat", namespace: "nilskaa@gmail.com", author: "Nils-Martin Skaanes", vid: "generic-thermostat-1") {
		capability "Temperature Measurement"
		capability "Thermostat"
		capability "Refresh"
		capability "Configuration"
		capability "energyMeter"
		capability "Lock"
		
		command "resetKWH"

        //raw: 01 0104 0301 00 03 0000 0003 0201 00
	fingerprint profileId: "0104", deviceId: "0301", inClusters: "0000, 0003, 0201", endpointId: "01"
	}

	// simulator metadata
	simulator { }

	tiles(scale: 2) {
    	multiAttributeTile(name:"ETS", type:"generic", width: 3, height: 2, canChangeIcon: true){
            tileAttribute("device.temperature", key: "PRIMARY_CONTROL") {
				attributeState "temperature", label:'${currentValue}°', unit:"C",  backgroundColors:[
            		[value: 31, color: "#153591"],
            		[value: 44, color: "#1e9cbb"], //for some reason all these values 
            		[value: 59, color: "#90d2a7"], //are related to Fahrenheit values?
            		[value: 74, color: "#44b621"], //Maybe only an issue for Android?
            		[value: 84, color: "#f1d801"],
            		[value: 95, color: "#d04e00"],
            		[value: 96, color: "#bc2323"]
        		]
			}
            tileAttribute("device.heatingSetpoint", key: "SLIDER_CONTROL", inactiveLabel: false, range:"(5..50)"){
			attributeState "heatingSetpoint", action:"thermostat.setHeatingSetpoint", backgroundColor:"#e86d13"
            }
            tileAttribute("device.temperatureMeasurement", key: "SECONDARY_CONTROL"){
			attributeState "temperatureMeasurement", label: '${currentValue}', backgroundColor:"#e86d13"
            }
        }
		
        valueTile("energy", "device.energy", inactiveLabel: false, decoration: "flat", width: 2, height: 1) {
			state "default", label:'${currentValue} kWh', action:"resetKWH", backgroundColor:"#ffffff"
		}
        standardTile("refresh", "device.refresh", inactiveLabel: false, decoration: "flat") {
			state "default", action:"refresh.refresh", icon:"st.secondary.refresh"
		}
        standardTile("configure", "device.configure", inactiveLabel: false, decoration: "flat") {
			state "configure", label:'', action:"configuration.configure", icon:"st.secondary.configure"
		}
        valueTile("mode", "device.mode", inactiveLabel: false, decoration: "flat", width: 2, height: 1) {
			state "mode", label:'${currentValue}', backgroundColor:"#ffffff"
		}
        standardTile("heating", "device.thermostatOperatingState", width:2, height:2, inactiveLabel: false, decoration: "flat") {
			state "default", label: "unknown", defaultState: true, icon: "st.unknown.unknown.unkown"
   			state "idle", label: "inaktiv", icon: "st.thermostat.heat", backgroundColor: "#ffffff"
 			state "heating", label: "aktiv", icon: "st.thermostat.heating", backgroundColor: "#e86d13" 
        }
        standardTile("lock", "device.lock", width:2, height:2, inactiveLabel: false) {
			state "default", label: "unknown", nextState:"...", defaultState: true, icon: "st.unknown.unknown.unkown"
   			state "locked", label: "Barne sikring på", action: "lock.unlock", nextState:"unlocked", icon: "", backgroundColor: "#00a0dc"
			state "unlocked", label: "Barne sikring av", action: "lock.lock", nextState:"locked", icon: "", backgroundColor: "#ffffff"
        }
        standardTile("identity", "device.identity", inactiveLabel: false, decoration: "flat", width: 4, height: 1) {
			state "default", label: '${currentValue}', backgroundColor:"#ffffff"
		}
	main "ETS"
	details(["ETS", "lock", "mode", "heating", "energy", "refresh", "identity", "configure"])
	}
}

preferences {
    section() {
        input "watts", "decimal", title: "Power consumption (W) e.g 1000", range: "1..*", required: false
    }
}

// parse events into attributes
def parse(String description) {
	log.debug "Parse description $description"
	List result = []
	def descMap = zigbee.parseDescriptionAsMap(description)
	log.debug "Desc Map: $descMap"
	List attrData = [[cluster: descMap.cluster ,attrId: descMap.attrId, value: descMap.value]]
	descMap.additionalAttrs.each {
	    attrData << [cluster: descMap.cluster, attrId: it.attrId, value: it.value]
	}
	attrData.each {
		def map = [:]
      
      //415 = Thermostat/Regulator
       if (it.cluster == "0201" && it.attrId == "0405") {
        	log.debug "OPERATING MODE=$it.value"
            state.operatingmode = it.value
            map.name = "mode"
            if (it.value == "01") map.value = "REGULATOR"
            else map.value = "THERMOSTAT"
        }
      
      	//415 = Boolean - heating/not heating
      	if (it.cluster == "0201" && it.attrId == "0415") {
		log.debug "HEATING=$it.value"
		map.name = "thermostatOperatingState"
        	if (it.value == "00") {map.value = "idle"}
		else if (it.value == "01") {map.value = "heating"}
        }
                
        //413 = Boolean - child lock
      	if (it.cluster == "0201" && it.attrId == "0413") {
		log.debug "CHILDLOCK=$it.value"
		map.name = "lock"
		if (it.value == "00") {map.value = "unlocked"}
		else if (it.value == "01") {map.value = "locked"}
        }
	
	//402 = zone text
      	if (it.cluster == "0201" && it.attrId == "0402") {
		log.debug "ZONETEXT=$it.value"
		map.name = "identity"
        	map.value = hexToAscii(it.value)
        }
      
        //403 = termostat modus. 00=luftføler, 01=gulvføler, 03=gulv vakt
        if (it.cluster == "0201" && it.attrId == "0403") {
		log.debug "CONFIG - Temperature sensor value=$it.value"
		state.tempsensor = it.value
            	map.name = "mode"
            	//check operating mode
            	if(state.operatingmode == "00"){
            		if (it.value == "00") { map.value = "Luftføler modus"} 
                	else if (it.value == "01") {map.value = "Gulvføler modus"} 
                	else if (it.value == "03") {map.value = "Maksvokter modus"}
            	}
            	else map.value = "Regulator"
            
          //0003 = minimum setpoint value    
        } else if (it.cluster == "0201" && it.attrId == "0003") {
        	log.debug "CONFIG - Minimum Setpoint value=${it.value} Hex"
			state.minsetpoint = Integer.parseInt(it.value,16) 
            
          //0004 = maximum setpoint value  
        } else if (it.cluster == "0201" && it.attrId == "0004") {
        	log.debug "CONFIG - Maximum Setpoint value=${it.value} Hex"
			state.maxsetpoint = Integer.parseInt(it.value,16)
            
          //0000 = measured air temperature  
        } else if (it.cluster == "0201" && it.attrId == "0000" && state.tempsensor == "00") {
			log.debug "MEASURED AIR TEMPERATURE - air sensor mode"
			map.name = "temperature"
            map.value = getTemperature(it.value)
		} else if (it.cluster == "0201" && it.attrId == "0000" && state.tempsensor == "01") {
			log.debug "MEASURED AIR TEMPERATURE - floor sensor mode"
			map.name = "temperatureMeasurement"
			map.value = getTemperature(it.value) + "° Luft temperatur"
		} else if (it.cluster == "0201" && it.attrId == "0000" && state.tempsensor == "03") {
			log.debug "MEASURED AIR TEMPERATURE - floor monitor mode"
			map.name = "temperature"
			map.value = getTemperature(it.value)
            
          //0012 = heating setpoint value  
        } else if (it.cluster == "0201" && it.attrId == "0012") {
			log.debug "HEATING SETPOINT"
			map.name = "heatingSetpoint"
			map.value = getTemperature(it.value)
            
          //409 = Measured floor temperature (probe)  
		} else if (it.cluster == "0201" && it.attrId == "0409" && state.tempsensor == "01") {
			log.debug "MEASURED FLOOR TEMPERATURE - floor sensor mode"
			map.name = "temperature"
			map.value = getTemperature(it.value)
        } else if (it.cluster == "0201" && it.attrId == "0409" && state.tempsensor == "03") {
			log.debug "MEASURED FLOOR TEMPERATURE - floor monitor mode"
			map.name = "temperatureMeasurement"
			map.value = getTemperature(it.value) + "° Gulv temperatur"
            
          //408 = power consumption pr 10 minuttes?  
        } else if (it.cluster == "0201" && it.attrId == "0408") {
			log.debug "Energy"
			def lrv = zigbee.convertHexToInt(it.value)
			state.lastKWh = (state.lastKWh ?: 0.0) + ((watts ?: 0.0) * (lrv/3.333)) / (60 * 60 * 1000)
			map.name = "energy"
            		map.value = (float) (Math.round(state.lastKWh * 10.0) / 10.0)
			map.unit = "kWh"
			result << createEvent(name: "power", value: (float) (watts ?: 0), unit: "W")
        } 
		if (map) {
			result << createEvent(map)
		}
		log.debug "Parse returned $map"
	}
	return result
}

def resetKWH() {
    log.trace "Resetting KWh"
    state.lastKWh = 0
    sendEvent(name: "energy", value: state.lastKWh, unit: "KWh")
}

def refresh() {
     log.debug "sending refresh command"
     def cmd = []
     cmd << "st rattr 0x${device.deviceNetworkId} 0x01 0x0201 0x0000"  // Air sensor temp (on unit)
     cmd << "delay 400"
     cmd << "st rattr 0x${device.deviceNetworkId} 0x01 0x0201 0x0012"  // Heating setpoint    
     cmd << "delay 400"
     cmd << "st rattr 0x${device.deviceNetworkId} 0x01 0x0201 0x0409"  // Floor sensor temp (external)
     cmd << "delay 400"
     cmd << "st rattr 0x${device.deviceNetworkId} 0x01 0x0201 0x0408"  // power consumption? 
     cmd << "delay 400"
     cmd << "st rattr 0x${device.deviceNetworkId} 0x01 0x0201 0x0413"  // childlock   
     cmd << "delay 400"
     cmd << "st rattr 0x${device.deviceNetworkId} 0x01 0x0201 0x0402"  // zonetext
     cmd << "delay 400"
     cmd << "st rattr 0x${device.deviceNetworkId} 0x01 0x0201 0x0405"  // Thermostat / Regulator mode
     cmd << "delay 400"	
     cmd << "st rattr 0x${device.deviceNetworkId} 0x01 0x0201 0x0403"  // Thermostat modes
     cmd << "delay 400"	
     return cmd
} 

def poll() {
	log.debug "Executing poll"
	refresh()
}

def getTemperature(value) {
	if (value != null) {
		def celsius = Integer.parseInt(value, 16) / 100
		if (location.temperatureScale == "C") {
			return celsius
		} else {
			return Math.round(celsiusToFahrenheit(celsius))
		}
	}
}

def setHeatingSetpoint(degrees) {
	def minHeatingSetpoint = state.minsetpoint
	def maxHeatingSetpoint = state.maxsetpoint
    
    log.debug "Minimum Setpoint value=${minHeatingSetpoint} & Maximum Setpoint value=${maxHeatingSetpoint}"
    
    //enforce limits of heatingSetpoint when in thermostat mode:
    if (state.operatingmode == "00"){
		if (degrees > maxHeatingSetpoint) {
			degrees = maxHeatingSetpoint
		} else if (degrees < minHeatingSetpoint) {
			degrees = minHeatingSetpoint
		}
    }
    
	if (degrees != null) {
		def temperatureScale = location.temperatureScale

		def degreesInteger = Math.round(degrees)
		log.debug "setHeatingSetpoint({$degreesInteger} ${temperatureScale})"
		sendEvent("name": "heatingSetpoint", "value": degreesInteger, "unit": temperatureScale)

		def celsius = (location.temperatureScale == "C") ? degreesInteger : (fahrenheitToCelsius(degreesInteger) as Double).round(2)
		"st wattr 0x${device.deviceNetworkId} 1 0x0201 0x0012 0x29 {" + hex(celsius * 100) + "}"
	}
}

def lock() {
	log.debug "Lock Child lock" 
	zigbee.writeAttribute(0x0201, 0x0413, 0x10, 01)
}

def unlock() {
	log.debug "Unlock Child lock"
	zigbee.writeAttribute(0x0201, 0x0413, 0x10, 00)
}

def configure() {
  log.debug "Set up reporting and get configuration attributes"
  sendEvent(name: "supportedThermostatModes", value: ["heat"])
  sendEvent(name: "thermostatMode", value: "heat")
  return zigbee.configureReporting(0x0201, 0x0000, 0x29, 0, 3600, 0x000A) +
  zigbee.configureReporting(0x0201, 0x0012, 0x29, 0, 3600, 0x0001) +
  zigbee.configureReporting(0x0201, 0x0409, 0x29, 0, 3600, 0x000A) +
  zigbee.configureReporting(0x0201, 0x0408, 0x21, 0, 3600, 0x0005) +
  zigbee.configureReporting(0x0201, 0x0415, 0x10, 0, 3600, null) +
  zigbee.configureReporting(0x0201, 0x0413, 0x10, 0, 3600, null) +
  zigbee.readAttribute(0x0201, 0x0403) +
  zigbee.readAttribute(0x0201, 0x0003) +
  zigbee.readAttribute(0x0201, 0x0004) +
  zigbee.readAttribute(0x0201, 0x0402) +
  zigbee.readAttribute(0x0201, 0x0405)
     
  //log.debug "binding to Thermostat cluster"
  //"zdo bind 0x${device.deviceNetworkId} 1 1 0x0201 {${device.zigbeeId}} {}"
} 

private hex(value) {
	new BigInteger(Math.round(value).toString()).toString(16)
}

private static String hexToAscii(String hexStr) {
    StringBuilder output = new StringBuilder("");
     
    for (int i = 0; i < hexStr.length(); i += 2) {
        String str = hexStr.substring(i, i + 2);
        output.append((char) Integer.parseInt(str, 16));
    }
     
    return output.toString();
}
