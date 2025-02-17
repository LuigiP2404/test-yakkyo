<template>
  <div>
    <div class="canvas-outer">
      <canvas width="600" height="350" id="canvas" class="canvas" :class="state"></canvas>
      <div class="overlay">
        <div class="location-details">
          <div class="city" id="city">{{ city }}</div>
          <div class="temp" id="temp">{{ temp }}</div>
        </div>
      </div>
    </div>
    <div class="sign">
      Coded by Luigi Paglionico
    </div>
  </div>
</template>

<script>
import axios from 'axios';

var cloud = function (options, imageAssets, windSpeed, randomRange, canvas, context) {
  this.type = 'cloud';
  this.img = options.img || imageAssets.cloud_02;
  this.imageAssets = imageAssets;
  this.windSpeed = windSpeed;
  this.randomRange = randomRange;
  this.canvas = canvas;
  this.context = context;

  this.width = this.img.width;
  this.height = this.img.height;

  var max = 10;
  this.xVelocity = (windSpeed - randomRange(0, max)) / 60;
  this.yVelocity = 0;

  this.x = options.x || randomRange(-100, canvas.width + 100)
  this.y = options.y || randomRange(0 - (this.height / 2), -60);
};

cloud.prototype.draw = function () {
  this.x += this.xVelocity;
  this.context.drawImage(this.img.image, 0, 0, this.img.width, this.img.height, this.x, this.y, this.img.width, this.img.height);

  if (this.xVelocity > 0) {
    
    if (this.x > this.canvas.width) {
      this.xVelocity = (this.windSpeed - this.randomRange(0, 10)) / 60;
      this.x = 0 - this.width;
    }
  }
  else {
    
    if (this.x < 0 - this.width) {
      this.xVelocity = (this.windSpeed - this.randomRange(0, 10)) / 60;
      this.x = this.canvas.width;
    }
  }


  return true;
};

var rainDrop = function (randomRange, canvas, context, rainColor, assets) {
  this.type = 'rain_drop';
  this.width = 3;
  this.randomRange = randomRange;
  this.context = context;
  this.rainColor = rainColor;
  this.assets = assets;
  this.canvas = canvas;
  this.height = this.randomRange(15, 25);

  this.x = randomRange(0, canvas.width);
  this.y = -10;

  this.xVelocity = 0;
  this.yVelocity = 8;
};

rainDrop.prototype.draw = function () {
  this.y += this.yVelocity;
  this.context.fillStyle = this.rainColor;
  this.context.fillRect(this.x, this.y, this.width, this.height);

  if (this.y > this.canvas.height) {
    
    if (Math.floor(Math.random() * 10) > 7) {
      for (var i = 0, n = this.randomRange(3, 5); i < n; i++) {
        this.assets.push(new splashDrop(this.x, this.randomRange, this.canvas, this.context));
      }
    }
    return false;
  }

  return true;
};



/*
|
| Splash drop particla
|
*/
var splashDrop = function (x, randomRange, canvas, context) {
  this.type = 'splash_drop';
  this.width = 3;
  this.height = 3;

  this.randomRange = randomRange;
  this.canvas = canvas;
  this.context = context;

  this.x = x;
  this.y = canvas.height;

  this.yVelocity = randomRange(-1, -3, false);
  this.xVelocity = randomRange(-2, 2, false);

  this.age = 0;
  this.maxAge = 30;
};

splashDrop.prototype.draw = function () {
  this.y += this.yVelocity;
  this.x += this.xVelocity;

  this.context.fillStyle = this.rainColor;
  this.context.fillRect(this.x, this.y, this.width, this.height);

  this.yVelocity += 0.1;

  this.age++;
  if (this.age > this.maxAge) {
    return false;
  }

  return true;
};




/*
|
| Snow flake particle
|
*/
var snowFlake = function (randomRange, canvas, context, windSpeed) {
  this.type = 'snow_flake';
  this.width = randomRange(10, 30);
  this.height = this.width;
  this.randomRange = randomRange;
  this.canvas = canvas;
  this.context = context;
  this.windSpeed = windSpeed;

  this.x = randomRange(-200, canvas.width + 200);
  this.y = -30;

  this.xVelocity = (windSpeed - randomRange(0, 10)) / 60;
  this.yVelocity = randomRange(.8, 1.4, false);

  this.opacity = randomRange(.3, .7, false);
  this.settleLength = 500;
  this.settled = 0;
};

snowFlake.prototype.draw = function () {
  this.y += this.yVelocity;
  this.x += this.xVelocity;

  this.context.beginPath();
  this.context.arc(this.x, this.y, this.width / 2, 0, 2 * Math.PI, false);
  this.context.fillStyle = 'rgba(255, 255, 255, ' + this.opacity + ')';
  this.context.fill();

  if (this.y > this.canvas.height) {
    this.xVelocity = 0;
    this.yVelocity = 0;
    this.settled++;

    if (this.settled > this.settleLength) {
      return false;
    }
  }

  return true;
};





/*
|
| Blowing leaf particle
|
*/
var blowingLeaf = function (randomRange, windSpeed, canvas, context, imageAssets) {
  this.type = 'blowing_leaf';
  this.randomRange = randomRange;
  this.width = randomRange(10, 20);
  this.height = this.width * 2.24;
  this.windSpeed = windSpeed;
  this.canvas = canvas;
  this.context = context;
  this.imageAssets = imageAssets;

  this.xVelocity = (windSpeed - randomRange(0, 20)) / 6;
  this.yVelocity = this.xVelocity / 6;

  this.rotation = Math.random() * 1;
  this.rotationVelocity = randomRange(-.06, .06, false);

  if (this.xVelocity > 0) {
    this.x = randomRange(-50, -100);
  }
  else {
    this.x = randomRange(canvas.width, canvas.width + 100);
  }

  this.gravity = randomRange(-0.06, 0.06, false);
  this.y = randomRange(canvas.height - canvas.height / 4, canvas.height);
  this.yDirectionChangeLength = randomRange(20, 100);
  this.yDirectionTravelled = 0;
};

blowingLeaf.prototype.draw = function () {
  this.context.save();

  this.x += this.xVelocity;
  this.y += this.yVelocity;

  this.yVelocity = this.yVelocity + this.gravity + -.01;

  this.yDirectionTravelled++;
  if (this.yDirectionTravelled > this.yDirectionChangeLength) {
    this.yDirectionTravelled = 0;
    this.gravity *= -1;
    this.yDirectionChangeLength = this.randomRange(20, 100);
  }

  this.rotation += this.rotationVelocity;

  var xOffset = this.width / 2;
  var yOffset = this.height / 2;

  this.context.translate(this.x + xOffset, this.y + yOffset);
  this.context.rotate(this.rotation);
  this.context.drawImage(this.imageAssets.leaf.image, 0, 0, 100, 224, 0 - xOffset, 0 - yOffset, this.width, this.height);

  this.context.restore();

  if (this.xVelocity > 0) {
    if (this.x > this.canvas.width) {
      return false;
    }
  }
  else {
    if (this.x < -50) {
      return false;
    }
  }
  return true;
};





/*
|
| Lightning particle
|
*/
var lightning = function (randomRange, canvas, context) {
  this.type = 'lightning';
  this.randomRange = randomRange;
  this.canvas = canvas;
  this.context = context;
  this.x = randomRange(0, canvas.width);
  this.age = 0;
  this.life = 20;
  this.drawFrom = 0;
  this.drawTo = 0;
  this.points = [
    [this.x, 0]
  ];
  this.totalPoints = 0;
  this.opacity = .7;

  this.flashed = false;
  this.flashOpacity = 0;

  var nextPointX = 0;
  var nextPointY = 0;
  while (nextPointY < canvas.height) {
    var lastPoint = this.points[this.points.length - 1];
    nextPointX = lastPoint[0] > this.x ? randomRange(this.x, this.x + 15) : randomRange(this.x + 15, this.x);
    nextPointY = lastPoint[1] + randomRange(10, 50);

    if (nextPointY > canvas.height) {
      nextPointY = canvas.height;
    }

    this.totalPoints++;
    this.points.push([nextPointX, nextPointY]);
  }
};

lightning.prototype.draw = function () {
  if (this.drawTo < this.points.length) {
    this.drawTo = this.drawTo + 2;
    if (this.drawTo > this.points.length) {
      this.drawTo = this.points.length;
    }
  }
  else {
    this.opacity = this.opacity - .02;

    if (!this.flashed) {
      this.flashed = true;
      this.flashOpacity = 1;
    }
  }

  if (this.opacity < 0) {
    return false;
  }

  if (this.flashOpacity > 0) {
    this.context.fillStyle = 'rgba(255, 255, 255, ' + this.flashOpacity + ')';
    this.context.fillRect(0, 0, this.canvas.width, this.canvas.height);
    this.flashOpacity = this.flashOpacity - .1;
  }

  this.context.beginPath();
  this.context.moveTo(this.points[this.drawFrom][0], this.points[this.drawFrom][1]);

  for (var i = this.drawFrom; i < this.drawTo; i++) {
    this.context.lineTo(this.points[i][0], this.points[i][1]);
  }

  this.context.strokeStyle = 'rgba(255, 255, 255, ' + this.opacity + ')';
  this.context.lineWidth = 3;
  this.context.stroke();

  return true;
};

export default {
  name: 'WeatherComponent',
  props: {
    msg: String
  },
  data() {
    return {
      city: null,
      temp: null,
      position: {
        lat: null,
        lng: null
      },
      condition: {
        clouds: false,
        lightning: false,
        rain: false,
        snow: false,
        wind: false
      },
      windSpeed: 0,
      windDirection: 0,
      state: null,
      rainColor: 'rgba(255, 255, 255, .4)',
      snowColor: '',
      imageAssetsLoaded: false,
      imageAssets: {
        'leaf': {
          fileName: 'weather_leaf.png'
        },
        'cloud_02': {
          fileName: 'weather_cloud_02.png',
          width: 1792,
          height: 276
        }
      },
      assets: [],
      timers: {},
      animationId: false,
      canvas: null,
      context: null
    }
  },
  methods: {
    fetchCurrentPosition() {
      if ("geolocation" in navigator) {
        navigator.geolocation.getCurrentPosition(
          async (position) => {
            const { latitude, longitude } = position.coords;
            this.position.lat = latitude;
            this.position.lng = longitude;
            this.getCityName();
            this.fetchWeather();
          },
          (error) => {
            console.error("There was an error while fetching the position", error.message);
          }
        );
      } else {
        console.log("Geolocation is not supported by this browser.");
      }
    },
    async fetchWeather() {
      let url = 'https://api.open-meteo.com/v1/forecast?current_weather=true';
      if (this.position.lat && this.position.lng) {
        url += `&latitude=${this.position.lat}&longitude=${this.position.lng}`;
        axios.get(url)
          .then((res) => {
            if (res.data.current_weather.temperature && res.data.current_weather_units.temperature) {
              this.temp = `${Math.round(res.data.current_weather.temperature)}${res.data.current_weather_units.temperature}`
            }
            if (res.data.current_weather.is_day !== null) {
              this.state = res.data.current_weather.is_day === 0 ? 'night' : 'day'
            }
            if (res.data.current_weather.winddirection && res.data.current_weather.windspeed) {
              this.windDirection = res.data.current_weather.winddirection;
              this.windSpeed = res.data.current_weather.windspeed;
            }
            if (res.data.current_weather.weathercode !== null) {
              const weatherCode = res.data.current_weather.weathercode;
              switch(weatherCode) {
                case 2:
                case 3:
                case 4:
                  this.condition.clouds = true;
                  break;
                case 6:
                case 7:
                case 8:
                case 9:
                  this.condition.rain = true;
                  this.condition.clouds = true;
                  break;
                case 10:
                  this.condition.lightning = true;
                  this.condition.rain = true;
                  this.condition.clouds = true;
                  break;
                case 11:
                case 12:
                case 13:
                  this.condition.snow = true;
                  this.condition.rain = true;
              }
            }
            this.preLoadImageAssets(() => {
              this.beginSpawning();
            })
          })
          .catch((err) => {
            console.error('There was an error during the fetch of the weather: ' + err);
          })
      } else {
        console.log('There is no value for lat and/or lng')
      }
    },
    getCityName() {
      const url = `https://nominatim.openstreetmap.org/reverse?format=json&lat=${this.position.lat}&lon=${this.position.lng}`;

      axios.get(url)
        .then(data => {
          if (data.data.address && data.data.address.city && data.data.address.country) {
            this.city = `${data.data.address.city}, ${data.data.address.country}`;
          } else {
            console.log("Location not found.");
          }
        })
        .catch(error => console.error("Error fetching location data: ", error));
    },
    randomRange(min, max, round) {
      round = round === undefined ? true : false;
      var val = Math.random() * (max - min) + min;
      return round ? Math.floor(val) : val;
    },
    preLoadImageAssets(callback) {
      var imageAssetsCount = 0;
      var imageAssetsLoadedCount = 0;

      if (this.imageAssetsLoaded) {
        if (callback) {
          callback();
        }
        return;
      }

      let loadedHandler = function () {
        imageAssetsLoadedCount++;
        if (imageAssetsLoadedCount === imageAssetsCount) {
          this.imageAssetsLoaded = true;
          if (callback) {
            callback();
          }
        }
      };

      for (var imageAssetName in this.imageAssets) {
        var imageAsset = this.imageAssets[imageAssetName];
        imageAssetsCount++;
        imageAsset.image = new Image();
        imageAsset.image.onload = loadedHandler;
        imageAsset.image.src = 'https://s3.amazonaws.com/gerwins/weather/' + imageAsset.fileName;
      }
    },
    beginSpawning() {
      if (this.condition.clouds) {
        this.assets.push(new cloud({ x: -400 }, this.imageAssets, this.windSpeed, this.randomRange, this.canvas, this.context));
        this.assets.push(new cloud({ x: 700 }, this.imageAssets, this.windSpeed, this.randomRange, this.canvas, this.context));
        this.assets.push(new cloud({ x: 1400 }, this.imageAssets, this.windSpeed, this.randomRange, this.canvas, this.context));
      }

      if (this.condition.rain) {
        const that = this;
        that.timers.rain = setInterval(function () {
          that.assets.push(new rainDrop(that.randomRange, that.canvas, that.context, that.rainColor, that.assets));
        }, 60);
      }

      if (this.condition.snow) {
        const that = this;
        that.timers.snow = setInterval(function () {
          that.assets.push(new snowFlake(that.randomRange, that.canvas, that.context, that.windSpeed));
        }, 250);
      }

      if (this.condition.wind) {
        const that = this;
        var spawnLeaves = function () {
          for (var i = 0, n = that.randomRange(0, 3); i < n; i++) {
            that.assets.push(new blowingLeaf(that.randomRange, that.windSpeed, that.canvas, that.context, that.imageAssets));
          }

          that.timers.wind = setTimeout(spawnLeaves, that.randomRange(500, 1500));
        };

        spawnLeaves();
      }

      if (this.condition.lightning) {
        const that = this;
        var spawnLightning = function () {
          var rand = that.randomRange(0, 10);
          if (rand > 7) {
            that.timers.secondFlash = setTimeout(function () {
              that.assets.push(new lightning(that.randomRange, that.canvas, that.context));
            }, 200);
          }
          that.assets.push(new lightning(that.randomRange, that.canvas, that.context));
          that.timers.lightning = setTimeout(spawnLightning, that.randomRange(500, 7000));
        };

        spawnLightning();
      }
      this.animate();
    },
    animate() {
      this.context.clearRect(0, 0, this.canvas.width, this.canvas.height);

      for (var i = 0, n = this.assets.length; i < n; i++) {
        if (!this.assets[i].draw()) {
          this.assets.splice(i, 1);
          n--;
          i--;
        }
      }

      this.animationId = window.requestAnimationFrame(this.animate);
    }
  },
   mounted() {
    this.canvas = document.getElementById('canvas');
    this.context = this.canvas.getContext('2d');
    this.fetchCurrentPosition();
  }
}
</script>

<style>
body {
  font-family: sans-serif;
  padding: 0;
  margin: 0;
}

.me {
  margin-top: 10px;
  padding-top: 10px;
  border-top: solid 1px #eee;
  text-align: center;
}

.me a {
  text-decoration: none;
  color: #26a69a;
}

.me .name {
  font-size: 16px;
}

.me .handle {
  font-size: 12px;
}

/* Overlay */
.canvas-outer {
  position: relative;
  max-width: 600px;
  overflow: hidden;
  margin: 0 auto;
}

.overlay {
  display: flex;
  position: absolute;
  align-items: center;
  top: 0px;
  left: 0px;
  width: 100%;
  height: 100%;

  color: #fff;
}

.location-details {
  text-align: center;
  width: 100%;
}

.city {
  font-size: 18px;
  text-align: center;
  text-shadow: 0px 1px 4px rgba(0, 0, 0, .1);
}

.temp {
  font-size: 100px;
  text-align: center;
  text-shadow: 0px 2px 7px rgba(0, 0, 0, .1);
}



/* Canvas */
.canvas {}

.night {
  background: linear-gradient(#140d15 0%, #212e3f 100%);
}

.day {
  background: linear-gradient(#90dffe 0%, #38a3d1 100%);
}

.sunrise {
  background: -webkit-linear-gradient(top, #82a0bc 0%, #aebebe 50%, #e0ceb0 80%, #ffc19c 100%);
  background: linear-gradient(top, #82a0bc 0%, #aebebe 50%, #e0ceb0 80%, #ffc19c 100%);
}

.sunset {
  background: -webkit-linear-gradient(top, #10070c 0%, #2e4557 40%, #91aba8 80%, #ddd8a1 100%);
  background: linear-gradient(top, #10070c 0%, #2e4557 40%, #91aba8 80%, #ddd8a1 100%);
}



/* Controls */
.controls {
  max-width: 600px;
  margin: 0 auto;
  padding-top: 20px;
}

@media (max-width: 600px) {
  .controls {
    padding: 20px 10px;
  }
}

.sign {
  margin-top: 2rem;
  font-size: 1rem;
}
</style>
