# Scalate support for SBT 0.1x.x
 
Integration for SBT that lets you generate sources for your Scalate templates and precompile them as part of the normal compilation process. This plugin is published to scala-tools.org.
 
## Usage

### Getting the plugin

Include the plugin in `project/plugins.sbt`:

For sbt 0.12.x, 0.13.x:

```scala
addSbtPlugin("com.mojolly.scalate" % "xsbt-scalate-generator" % "0.4.2")
```

For sbt 0.11.3:

```scala
resolvers += Resolver.url("sbt-plugin-releases",
  new URL("http://scalasbt.artifactoryonline.com/scalasbt/sbt-plugin-releases/"))(
    Resolver.ivyStylePatterns)

addSbtPlugin("com.mojolly.scalate" % "xsbt-scalate-generator" % "0.2.0")
```

for sbt 0.11.2: (maven central)

```scala
libraryDependencies <+= sbtVersion(v => "com.mojolly.scalate" %% "xsbt-scalate-generator" % (v + "-0.1.6"))
```

or as a git dependency in `project/project/build.scala`:

```scala
import sbt._
import Keys._

object PluginsBuild extends Build {
  lazy val root = Project("plugins", file(".")) dependsOn (scalateGenerate) settings (scalacOptions += "-deprecation")
  lazy val scalateGenerate = ProjectRef(uri("git://github.com/mojolly/xsbt-scalate-generate.git"), "xsbt-scalate-generator")
}
```

### Configuring the plugin

Configure the plugin in `build.sbt`:

```scala

import ScalateKeys._

seq(scalateSettings:_*)
      
// Scalate Precompilation and Bindings
scalateTemplateConfig in Compile <<= (sourceDirectory in Compile){ base =>
  Seq(
    TemplateConfig(
      base / "webapp" / "WEB-INF" / "webTmpl",
      Seq(
        "import org.myapp.scalate.Helpers._",
        "import org.myapp.model._",
        "import net.liftweb.common._",
        "import org.joda.time._",
        "import org.scalatra.UrlGenerator"
      ),
      Seq(
        Binding("context", "_root_.org.scalatra.scalate.ScalatraRenderContext", importMembers = true, isImplicit = true),
        Binding("messageTranslatorModel", "org.myapp.model.mongo.MessageTranslator", importMembers = true, isImplicit = true, defaultValue = null),
        Binding("userSession", "org.myapp.auth.UserSession", importMembers = true, defaultValue = null),
        Binding("env", "org.myapp.util.Environment")
      ),
      Some("webTmpl")
    ),
    TemplateConfig(
      base / "webapp" / "WEB-INF" / "mailTmpl",
      Seq(
        "import org.myapp.scalate.Helpers._",
        "import org.myapp.model._",
        "import net.liftweb.common._",
        "import org.joda.time._"
      ),
      Seq(
        Binding("i18n", "org.myapp.model.mongo.MessageTranslator", true, isImplicit = true, defaultValue = null),
        Binding("user", "User", false, defaultValue = null),
        Binding("config", "com.typesafe.config.Config", false, defaultValue = null),
        Binding("assets", "org.myapp.model.mongo.fields.AssetPaths", false, isImplicit = true, defaultValue = null),
        Binding("geonames", "org.myapp.model.Geonames", false, isImplicit = true, defaultValue = null)
      ),
      Some("mailTmpl")
    )
  )
)

```

### Trigger recompilation on save

From version 0.2.2 onwards the plugin detects when sources are changed and will trigger a recompilation.
Older versions can add this to their build.sbt:

```scala
watchSources <++= (scalateTemplateDirectory in Compile) map (d => (d ** "*").get)
```

### To use multiiple template directories with scalatra you'll need to make some changes too: 

```scala
trait YourScalateSupport extends ScalateSupport {
 
  override protected def defaultTemplatePath: List[String] = List("/webTmpl/views")
 
  override protected def createTemplateEngine(config: ConfigT) = {
    val engine = super.createTemplateEngine(config)
 
    engine.layoutStrategy = new DefaultLayoutStrategy(engine,
      TemplateEngine.templateTypes.map("/webTmpl/layouts/default." + _): _*)
 
    engine.packagePrefix = "webTmpl"
    engine
  }
 
}
```


## Patches

Patches are gladly accepted from their original author. Along with any patches, please state that the patch is your original work and that you license the work to the *xsbt-scalate-generate* project under the MIT License.
 
## License
 
MIT licensed. Check the [LICENSE](https://raw.github.com/backchatio/xsbt-scalate-generate/master/LICENSE) file.
