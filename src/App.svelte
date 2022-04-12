<script>
  import { version } from './version.js'
  import * as config from './config';

  let client = mqtt.connect(`ws://${config.MQTT_BROKER}`,
    {clientId: config.CLIENT_ID});

  const sonoff = config.SONOFF_SWITCH;

  client.on('connect', function () {
    console.log("connected");
    const power_topic = `stat/${sonoff}/POWER`
    client.subscribe(power_topic, function (err) {
      if (err) {
        console.log(`mqtt connect error: ${err}`)
      } else {
        console.log('subscribed with topic: ', power_topic);
      }
    })
  })

  client.on('message', function (topic, message) {
    if (topic.includes(`${sonoff}/POWER`)) {
      const state = message.toString() === 'ON';
      if (state){
        const cmnd_topic = `cmnd/${sonoff}/power`;
        console.log('mqtt message: sonoff is ON');
        setTimeout(() => {
          client.publish(cmnd_topic, 'OFF', {retain: false})
          document.querySelector('button').style.backgroundColor='green';
        }, config.BUTTON_PUSHED_TIME);
     }
    }
  })

  const start_motor = () => {
    console.log('start_motor: button pushed');
    const cmnd_topic = `cmnd/${sonoff}/power`;
    client.publish(cmnd_topic, 'ON', {retain: false});
    document.querySelector('button').style.backgroundColor='red';
    setTimeout(() => {
      client.publish(cmnd_topic, 'OFF', {retain: false})
      document.querySelector('button').style.backgroundColor='green';
    }, config.BUTTON_PUSHED_TIME);
  }
</script>

<main>
  <div><img src="garage-door.png" width="200px"></div>
  <button on:click={start_motor} >Hier klikken</button>
</main>

<footer class="bg-gray-300 p-8 box-border">
  <p>{@html version}</p>
</footer>

<style>
	main {
		text-align: center;
		padding: 1em;
		max-width: 240px;
		margin: 0 auto;
	}

	h1 {
		color: #ff3e00;
		text-transform: uppercase;
		font-size: 4em;
		font-weight: 100;
	}

	@media (min-width: 640px) {
		main {
			max-width: none;
		}
	}

  button {
      color: white;
      background-color: green;
  }
</style>
