#!/usr/bin/env node

/**
 * Simple cli script to manage bash profiles JIT.
 *
 * @version 1.0.0
 * @author Jared Allard <jaredallard@outlook.com>
 * @license MIT
 **/

'use strict';

// version consts
const VERSION="1.0.0";

// node.js scripts
const path   = require('path');
const fs     = require('fs');
const spawn  = require('child_process').spawnSync;

const mkdirp = require('mkdirp');

// dynamic "settings"
let profiles_loc = '{{HOME}}/.at';

profiles_loc = profiles_loc.replace('{{HOME}}',
  process.env[(process.platform == 'win32') ? 'USERPROFILE' : 'HOME']);

let profilectl = path.join(profiles_loc, 'at.json');
if(!fs.existsSync(profilectl)) {
    fs.writeFileSync(profilectl, JSON.stringify({
      active: null,
      default: null
    }), 'utf8');
}

let profile = require(profilectl);

// verify the dir exists
mkdirp.sync(profiles_loc);

/**
 * Create a new profile
 **/
const makeNewProfile = function(profile) {
  let profile_loc = path.join(profiles_loc, profile);

  if(fs.exists(profile_loc)) {
    return false;
  }

  // mkdir -p the directory.
  mkdirp.sync(profile_loc);

  return true;
}

/**
 * Check if a profile is valid
 *
 * @param {String} profile - profile name
 * @return {Bool} success
 **/
const profileIsValid = function(profile) {
  let profile_loc  = path.join(profiles_loc, profile);
  let profile_json = path.join(profile_loc, 'profile.json');

  if(!fs.existsSync(profile_loc)) {
    return 'Profile \''+profile+'\' doesn\'t exist';
  }

  if(!fs.existsSync(profile_json)) {
    return 'No profile.json! Did you create one?';
  }

  return true;
}

/**
 * Execute a profile
 **/
const executeProfile = function(profile) {
  let profile_loc  = path.join(profiles_loc, profile);
  let profile_json = path.join(profile_loc, 'profile.json');

  let res = profileIsValid(profile)
  if(res !== true) {
    return res;
  }

  let oprof = require(profile_json);

  oprof.actions.forEach(function(v) {
    let name = v.command,
        opts = v.opts;

    // conditions
    if(name === 'exists') {
      if(fs.existsSync(opts[0])) {
        name = v.then.command;
        opts = v.then.opts;
      }
    }

    // TODO: Array to contain all possible command types &/or inst
    if(name === 'set') {
      console.log('export', opts[0]+'='+opts[1]);
    } else if(name === 'mv') {
      moveFile(opts[0], opts[1]);
    } else if(name === 'exec') {
      spawn(opts[0]);
    } else if(name === 'cp') {
      copyFile(opts[0], opts[1]);
    } else {
      console.log(name);
    }
  })

  return true;
}

/**
 * Display a ln -v like arrow thing.
 **/
const fancyArrow = function(f, t) {
  let fancyTemplate = "'{{f}}' -> '{{t}}'";
  fancyTemplate = fancyTemplate.replace('{{f}}', f);
  fancyTemplate = fancyTemplate.replace('{{t}}', t);

  console.log(fancyTemplate)
}

/**
 * Move a file.
 **/
const moveFile = function(from, to) {
  fancyArrow(from, to);

  let res,
      status;

  try {
    res = fs.readFileSync(from, 'utf8');
    status = fs.writeFileSync(to, res, 'utf8');
  } catch(err) {
    console.log('Failed to execute move.');
    process.exit(1);
  }
}

/**
 * Save the profile object
 **/
const saveState = function() {
  if(fs.writeFileSync(profilectl, JSON.stringify(profile), 'utf8')) {
    return true;
  } else {
    return false;
  }
}

// check opts
let opt1 = process.argv[2],
    opt2 = process.argv[3],
    opt3 = process.argv[4];

if(opt1 === 'new') {
  if(!makeNewProfile(opt2)) {
    console.log('Failed to create new profile \''+opt2+'\'')
  } else {
    console.log('Profile successfully created');
  }
} else if(opt1 === 'list') {
  console.log(fs.readdirSync(profiles_loc).toString('ascii').replace(/\,/g, '\n'));
} else if(opt1 === 'active') {
  console.log(profile.active);
} else if(opt1 === 'set') {
  if(opt2 === 'default') {
    let res = profileIsValid(opt3);
    if(res !== true) {
      return res;
    }

    profile.default = opt3;
    saveState();

    console.log('Profile \''+opt3+'\' is now the default');
  } else {
    console.log('Cannot set, not supported.')
  }
} else if(opt1 === undefined || opt1 === 'help') {
  console.log('at - profile switcher\n')
  console.log('USAGE: at [cmd] [profile]');
  console.log('       at [profile]\n')
  console.log('COMMANDS:')
  console.log('       help    - display this page');
  console.log('       set     - set a constant (i.e default)');
  console.log('       list    - list all profiles');
  console.log('       active  - print out the active profile');
  console.log('');
  console.log('Report bugs to https://github.com/jaredallard/at');
} else {
  if(profile.active === opt1) {
    console.log('Profile \''+opt1+'\' is already active');
    process.exit(0);
  }

  // try to execute profile
  let res = executeProfile(opt1);
  if(res !== true) {
    console.log(res);
  } else {
    profile.active = opt1;
    saveState();

    console.log('profile executed.')
  }
}
