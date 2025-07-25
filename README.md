# cTrader Console Docker Image

## About this Image

This official container images for cTrader Console on Linux for Docker Engine.

## How to use this Image

You can run cTrader console by pulling this image and then running it.

Pull:

```
docker pull ghcr.io/spotware/ctrader-console:5.4
```

Run:

```
docker run -d -it --name ctrader.console.run.mybot --mount type=bind,src=/cAlgo/Robots,dst=/mnt/Robots -e CTID='mycid' -e PWD-FILE='/mnt/Robots/ctrader-cli.pwd' -e ACCOUNT='9102302' -e SYMBOL='EURUSD' -e PERIOD='H1' ghcr.io/spotware/ctrader-console:5.4 run "/mnt/Robots/My bot.algo" --environment-variables
```

We use Docker mount to let the container have access to a directory so it will be able to run the cBot algo file.

In above command we use:

* `/cAlgo/Robots`: Path in host that will be mounted to container
* `/mnt/Robots`: Container mount path
* `CTID='mycid'`: Your cId username
* `/mnt/Robots/ctrader-cli.pwd`: Your cID password file path according to mount path
* `ACCOUNT='9102302'`: Your trading account number
* `SYMBOL='EURUSD'`: Symbol that cBot will run on
* `PERIOD='H1'`: Timeframe that cBot will rub on
* `ghcr.io/spotware/ctrader-console:5.4`: Container image name that you pulled, change the tag if you pulled some specific version
* `/mnt/Robots/My bot.algo`: Your cBot algo file path according to mount path
* `environment-variables`: We use the to let cTrader console know it should use environment variables for getting its parameters

You can also use this image to build another docker image with your cBot algo and cID password file that can be run easily anywhere. 

You can use all of the cTrader console commands like backtest, for more regarding available features in cTrader Console please check the documentation.

## Tags

We use latest for latets available version, to use specific version you can pull it by using the version as tag, ex:

```
docker pull ghcr.io/spotware/ctrader-console:5.4
```

## Useful Links

[cTrader CLI Documentation](https://help.ctrader.com/ctrader-algo/ctrader-cli/)
