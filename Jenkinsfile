/*
 * See the NOTICE file distributed with this work for additional
 * information regarding copyright ownership.
 *
 * This is free software; you can redistribute it and/or modify it
 * under the terms of the GNU Lesser General Public License as
 * published by the Free Software Foundation; either version 2.1 of
 * the License, or (at your option) any later version.
 *
 * This software is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU
 * Lesser General Public License for more details.
 *
 * You should have received a copy of the GNU Lesser General Public
 * License along with this software; if not, write to the Free
 * Software Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA
 * 02110-1301 USA, or see the FSF site: http://www.fsf.org.
 */

// It's assumed that Jenkins has been configured to implicitly load the vars/*.groovy libraries.
// Note that the version used is the one defined in Jenkins but it can be overridden as follows:
// @Library("XWiki@<branch, tag, sha1>") _
// See https://github.com/jenkinsci/workflow-cps-global-lib-plugin for details.

def globalMavenOpts = '-Xmx1536m -XX:MaxPermSize=512m -Xms256m'

// Allow 2 commons builds at the same time to leave some agent space for other jobs
stage name: 'Commons Builds', concurrency: 2

parallel(
  "main": {
    node {
      // Build, skipping checkstyle & revapi so that the result of the build can be sent as fast as possible
      // to the dev. However note that in // we start a build with the quality profile that checks checkstyle
      // revapi and more.
      // Configures the snapshot extension repository in XWiki in the generated distributions to make it easy for
      // developers to install snapshot extensions when they do manual tests.
      xwikiBuild {
        mavenOpts = globalMavenOpts
        goals = 'clean deploy'
        profiles = 'legacy,integration-tests'
        properties = '-Dxwiki.checkstyle.skip=true -Dxwiki.surefire.captureconsole.skip=true -Dxwiki.revapi.skip=true'
      }
    }
  },
  "testrelease": {
    node {
      // Simulate a release and verify all is fine.
      xwikiBuild {
        mavenOpts = globalMavenOpts
        goals = 'clean install'
        profiles = 'legacy,integration-tests'
        properties = '-DskipTests -DperformRelease=true -Dgpg.skip=true -Dxwiki.checkstyle.skip=true'
      }
    }
  },
  "quality": {
    node {
      // Run the quality checks
      xwikiBuild {
        mavenOpts = globalMavenOpts
        goals = 'clean install jacoco:report'
        profiles = 'quality,legacy'
      }
    }
  },
  "checkstyle": {
    node {
      // Build with checkstyle. Make sure "mvn checkstyle:check" passes so that we don't cause false positive on
      // Checkstyle style. This is for the Checkstyle project itself so that they can verify that when they bring
      // changes to Checkstyle, there's no regression to the XWiki build.
      xwikiBuild {
        mavenOpts = globalMavenOpts
        goals = 'clean test-compile checkstyle:check'
        profiles = 'legacy'
      }
    }
  }
)