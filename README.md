# validator
A scala light library to create reusable validator
```
  import com.wzhi.tools.validate.nel.str.PredefinedValidators._
  import com.wzhi.tools.validate.nel.str.ValidatorStrErr
  import ValidatorStrErr._
  import ValidatorStrErr.strNelValidator._

  // Let's define some data classes below and give a sample data to verify.
  case class Profile(id: Long, name: String, age: Int, sex: String, email: Option[String])
  case class Team(profiles: List[Profile])

  // Let's create some business required validator with predefined validator create tools.
  // Please check all useful tools under PredefinedValidators
  val validSex = valueIn("male", "female", "gay", "lesbian")
  val acceptableAge = validate[Int](age => age > 20 && age < 40) otherwise "exceeds valid age scope."

  // >>+ means previous and next validator must pass, if any of them failed, skip the rest of validators.
  // <+> mean validate both validator, return all errors.
  // <<+ means any of the validator success will be treated as valid.
  val acceptableName = notNull[String] >>+ notEmptyStr >>+ strLengthLimit(23)

  // Now let's use validators to construct more complex validator to validate our data model.
  val profile = acceptableName.lift[Profile](_.name, "name") <+>
                validSex.lift(_.sex, "sex") <+>
                acceptableAge.lift(_.age, "age") <+>
                email.forOption.lift(_.email, "email")

  // to validate a list of same type data, we need a indicator to indicate which row has problem.
  val indicator = (p: Profile) => s"${p.id}'s "
  val teamValidator = profile.forList(indicator).lift[Team](_.profiles, "profiles")

  // Now let's create some test data
  val team = Team(
    Profile(0, null, 12, "male", Option("test@gmail.com"))::
      Profile(1, "", 12, "wrong", Option("@gmail.com"))::
      Profile(2, "a super long name which doesn't make sense", 80, "wrong", Option("test@gmail.com"))::
      Profile(3, "Jenny", 27, "female", Option("test@gmail.com"))::
      Nil
  )

  println(teamValidator validate team)

//  Invalid(NonEmptyList(
//    profiles/0's /name/null cannot be null.,
//    profiles/0's /age/12 exceeds valid age scope.,
//    profiles/1's /name/ cannot be empty.,
//    profiles/1's /sex/wrong not in male female gay lesbian,
//    profiles/1's /age/12 exceeds valid age scope.,
//    profiles/1's /email/@gmail.com not a valid email address,
//    profiles/2's /name/a super long name which doesn't make sense length exceeds 23.,
//    profiles/2's /sex/wrong not in ArraySeq(male, female, gay, lesbian),
//    profiles/2's /age/80 exceeds valid age scope.
//  ))

```
